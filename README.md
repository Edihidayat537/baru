
<?php
$response = LoginShopee('username','password');
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
echo " \033[1;32m[\033[1;35m?\033[1;32m] Otp => \033[1;33m";
	$otpp = trim(fgets(STDIN));
	
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
