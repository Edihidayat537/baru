shopeebot
<?php
$response = LoginShopee('u3cl8cv3g3','Riyadi0410');
echo $response;
echo " \033[1;32m[\033[1;35m?\033[1;32m] Otp => \033[1;33m";
	$otpp = trim(fgets(STDIN));

function LoginShopee($SP_Username,$SP_Pass) {

	//Inisialisasi umum untuk semua permintaan di bawah
	$ch=curl_init();
	curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
	curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
 	curl_setopt($ch, CURLOPT_USERAGENT, "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:70.0) Gecko/20100101 Firefox/70.0");
	$cookie_jar = 'cookie_shopee.txt';
 	curl_file_create($cookie_jar);	
	curl_setopt($ch, CURLOPT_COOKIEJAR, $cookie_jar);
	curl_setopt($ch, CURLOPT_COOKIEFILE, $cookie_jar);


	// Permintaan untuk mendapatkan cookie asli
	curl_setopt($ch, CURLOPT_URL, "https://shopee.co.id/api/v0/buyer/login/");
	curl_exec($ch);
	// echo file_get_contents($cookie_jar) .'<br><br>';


	//Inisialisasi data input untuk permintaan Login
	$csrf_token = csrftoken();

 	$header = array(
		'x-csrftoken: '. $csrf_token,
		'x-requested-with: XMLHttpRequest',
		'referer: https://shopee.co.id/api/v0/buyer/login/',
   	 );

	 $data = array(
		"login_key" => $SP_Username,
		"login_type" => "username",
		"password_hash" => CryptPass($SP_Pass),
		"captcha" => "",
		"remember_me" => "true"
	);
	$data = http_build_query($data);	

	//Request login
	curl_setopt($ch, CURLOPT_URL, "https://shopee.co.id/api/v0/buyer/login/login_post/"); 
	curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
	curl_setopt($ch, CURLOPT_COOKIE, "csrftoken=" . $csrf_token);
	curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
	curl_setopt($ch, CURLOPT_POSTFIELDSIZE, strlen($data));
	curl_setopt($ch, CURLOPT_POST, 1);
	curl_setopt($ch, CURLOPT_HEADER, 1);
	$response = curl_exec($ch);
	echo $response;
	
	// Permintaan untuk mendapatkan informasi akun setelah berhasil login
	curl_setopt($ch, CURLOPT_URL, "https://banhang.shopee.co.id/api/v1/login/");
	$response = curl_exec($ch);
	return $response;
	curl_close($ch);
}
function CryptPass($pass){
   return hash("sha256", md5($pass));
}
function csrftoken() {
    $karakter = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    $PanjangKarakter = strlen($karakter);
    $acakString = '';
    for ($i = 0; $i < 32; $i++) {
        $acakString .= $karakter[rand(0, $PanjangKarakter - 1)];
    }
    return $acakString;
}
?>

public function getAuth(){
/**
* @param ShopApiShopee $model
*/
$httpType = ((isset($_SERVER['https://shopee.co.id']) && $_SERVER['https:shopee.co.id'] == 'on') || (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https://shopee.co.id'))
? 'https://shopee.co.id' : 'http://shopee.co.id';
$redirectUrl = $httpType . $_SERVER['HTTP_HOST'] . '/';
$token = hash('sha256', $this->_ShopApiShopee->shopee_key . $redirectUrl);
$url = self::SHOPEE_GET_AUTH_URL
. '?id=' . $this->_ShopApiShopee->shopee_shop_id
. '&token=' . $token
. '&redirect=' . $redirectUrl;
header("Location: " . $url);
$this->_ShopApiShopee->token = $token;
if (!$this->_ShopApiShopee->save()) {
return self::fail(self::CODE_SAVE_ERROR,serialize($this->_ShopApiShopee->getErrors()));
}
}
static function getOrderList($shopApiShopee, $days = 1, $offsetDay = 0){
if(!empty($shopApiShopee) && $shopApiShopee instanceof ShopApiShopee )
{
$to_time_stamp = time() - $offsetDay * 86400;
$from_time_stamp = $to_time_stamp - 86400 * $days;
$arr = array(
'partner_id' => (int)$shopApiShopee->shopee_partner_id,
'shopid' => (int)$shopApiShopee->shopee_shop_id,
'timestamp' => time(),
'create_time_from' => $from_time_stamp,
'create_time_to' => $to_time_stamp,
'pagination_offset' => 0,
'pagination_entries_per_page' => 100
);
$result = self::_getCurlResponse($shopApiShopee, self::SHOPEE_GET_ORDER_LIST_URL, $arr);
return $result;
}
else
{
return self::fail(self::CODE_NO_FIND, 'shopee param invalid:' . serialize($shopApiShopee));
}
}
private static function _getCurlResponse($shopApiShopee, $url, $arr){
$arr = json_encode($arr);
$contentLength = strlen($arr);
$strConcat = $url.'|'.$arr;
$authorizationKey = hash_hmac('sha256', $strConcat, $shopApiShopee->shopee_key);
$header = array(
'Content-Type:application/json',
"Content-Length:" . $contentLength,
'Authorization:' . $authorizationKey
);
$params = array(
'base_uri' => $url,
'headers' => $header,
'verify' => false,
'body' => $arr
);
return self::_curl($params);
}
private static function _curl($params){
$ch = curl_init($params['base_uri']);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);
curl_setopt($ch, CURLOPT_HTTPHEADER, $params['headers']);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
// curl_setopt($ch, CURLOPT_HEADER, TRUE); 返回打印信息中包含头文件
curl_setopt($ch, CURLOPT_POST, TRUE);
curl_setopt($ch, CURLOPT_POSTFIELDS, $params['body']);
$str = curl_exec($ch);
$status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);
if( $status === self::CODE_SUCCESS ){
return self::success(json_decode($str,true), self::CODE_SUCCESS);
}else{
return self::fail(json_decode($str,true), $status);
}
}
