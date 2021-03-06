# Memcached

Memcached is an open source, high performance, distributed memory caching system intended to speed up dynamic web applications by reducing the database load.

It is a key/value dictionary of strings, objetcs, etc., stored in memory, resulting from database calls, API calls
, or page rendering.

Memcached was developed by Brad Fitzpatrick in 2003, it is now being used by Netlog, Facebook, Flickr, Wikipedia, Twitter, and Youtube among others.

Memcached on its own is simply a key/value store daemon. Installing it does not automatically accelerate or cache any of your data:

Your applications need to be programmed to utilize the service. Applications will need clients, of which there should ne ones readily available for your language of choice.

The key features of memcached are as follows:

- open soure
- memcached server is a big hash table
- it significantly reduces the database load
- it is perfectly efficient for websites with high database load
- it is a client server application over TCP or UDP
- it is distributed under BSD license

Memcached is not:

- a persistent data store
- a databse
- application specific
- a large object cache
- fault tolerant or highly available

## Installation

```
# Installation sur Debian et Ubuntu
apt-get install memcached
 
# Installation sur Redhat et Fedora
yum install memcached
 
# Installation sur Mac OS X (avec Homebrew)
brew install memcached
 
# Installation depuis la source
get http://memcached.org/latest
tar -zxvf memcached-1.x.x.tar.gz
cd memcached-1.x.x
./configure
make && make test
sudo make install
```

```
 apt-get install memcached
Lecture des listes de paquets... Fait
Construction de l'arbre des dépendances
Lecture des informations d'état... Fait
Les paquets suivants ont été installés automatiquement et ne sont plus nécessair                                                                                                             es :
  gyp libc-ares-dev libc-ares2 libjs-node-uuid libv8-3.14-dev libv8-3.14.5
Veuillez utiliser « apt-get autoremove » pour les supprimer.
Paquets suggérés :
  libcache-memcached-perl libmemcached
Les NOUVEAUX paquets suivants seront installés :
  memcached
0 mis à jour, 1 nouvellement installés, 0 à enlever et 178 non mis à jour.
Il est nécessaire de prendre 67,3 ko dans les archives.
Après cette opération, 230 ko d'espace disque supplémentaires seront utilisés.
Réception de : 1 http://fr.archive.ubuntu.com/ubuntu/ trusty-updates/main memcac
hed amd64 1.4.14-0ubuntu9.3 [67,3 kB]
67,3 ko réceptionnés en 0s (275 ko/s)
Sélection du paquet memcached précédemment désélectionné.
(Lecture de la base de données... 199572 fichiers et répertoires déjà installés.                                                                                                             )
Préparation du dépaquetage de .../memcached_1.4.14-0ubuntu9.3_amd64.deb ...
Dépaquetage de memcached (1.4.14-0ubuntu9.3) ...
Traitement des actions différées (« triggers ») pour ureadahead (0.100.0-16) ...
ureadahead will be reprofiled on next reboot
Traitement des actions différées (« triggers ») pour man-db (2.6.7.1-1ubuntu1) .                                                                                                             ..
Paramétrage de memcached (1.4.14-0ubuntu9.3) ...
Starting memcached: memcached.
Traitement des actions différées (« triggers ») pour ureadahead (0.100.0-16) ...

```

## Confirming memcached installation

By default, Memcached uses port 11211. To confirm if Memcached is installed or not, you need to run the command given below:

```
# ps -aux | grep memcached
memcache 12168  0.0  0.0  63264  2552 ?        Sl   23:34   0:00 /usr/bin/memcached -m 64 -p 11211 -u memcache -l 127.0.0.1

```

To run Memcached server on a different port, execute the command given below. This command starts the server on the TCP port 22222 and listens on the UDP port 11111 as a daemon process.

```
# ps -aux | grep memcached
memcache 12168  0.0  0.0  63264  2552 ?        Sl   23:34   0:00 /usr/bin/memcached -m 64 -p 11211 -u memcache -l 127.0.0.1
memcache 12718  0.0  0.0 325408  2264 ?        Ssl  23:37   0:00 memcached -p 22222 -U 11111 -u memcache -d

```

You can run multiple instances of memcached server through a single installation.

let's try a telnet connection to the server,

Remember, the set storage command needs some values like:
- flags (32-bit unsigned integer that the server stores with data provided by the user and returns along with the data when the item is retrieved), 
- time to live or TTL in second (0 means no delay)
- size in byte

```
# telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
set mykey1 0 15 5
ABCDE
STORED
get mykey1
END
set mykey1 0 30 5
ABCDE
STORED
get mykey1
VALUE mykey1 0 5
ABCDE
END
get mykey1
VALUE mykey1 0 5
ABCDE
END
get mykey1
END
set badDataChunkForTest 0 30 5 ABCDE
memcached
CLIENT_ERROR bad data chunk
ERROR

```

Here STORED indicates a success and ERROR an incorrect syntax or error while saving data.

With the command `add`, if the key already exists, then it gives the output NOT_STORED (like insert)
With the command `replace`, if the key does not exist, then it also gives the output NOT_STORED (like update)

Other storage commands are: Append, Prepend and CAS (stands for Check And Set)

## Testing memcached on PHP

you need to install php5-memcached and or php7.0-memcached and restart your apache server.

```
<?php

class User
{
    private $name;
    private $password;
    private $role;
    public function __construct($name, $password, $role)
    {
        $this->$name = $name;
        $this->$password = $password;
        $this->$role = $role;

    }
}

$user = new User('John Doe', 'superSecretPassword', 'SuperMan');

function AccessDataBase(){
    echo "Simulate an access to the database! Your DB load is up ..." . PHP_EOL;
        sleep(mt_rand(1,10));
}
$m = new Memcached();
$m->addServer('localhost', 11211);

for($i = 0; $i < 10000; $i++){
    if($m->get('user') !== FALSE){
        echo "$i: getting data from memcached...";
        var_dump($m->get('user'));
//      sleep(mt_rand(1,10));
    } else {
        echo "$i: getting data from database...";
        AccessDataBase();
        //var_dump($user);
        $m->set('user', $user, time() + 3);
    }
}
?>

```

## Memcached on CakePHP

https://book.cakephp.org/3.0/en/core-libraries/caching.html

Data Source Name is basically the same as Symfony

## Memcached on Symfony

https://symfony.com/doc/current/components/cache/adapters/memcached_adapter.html

## Memcached on Laravel

https://laravel.com/docs/5.7/cache

## Memcached on Zend

https://framework.zend.com/manual/2.3/en/modules/zend.cache.storage.adapter.html#the-memcached-adapter

## Memcached on CodeIgniter

https://www.codeigniter.com/userguide3/libraries/caching.html#memcached-caching