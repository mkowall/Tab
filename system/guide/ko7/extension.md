# Transparent Class Extension

The [cascading filesystem](files) allows transparent class extension. For instance, the class [Cookie] is defined in `SYSPATH/classes/Cookie.php` as:

    class Cookie extends KO7_Cookie {}

The default Koseven classes, and many extensions, use this definition so that almost all classes can be extended. You extend any class transparently, by defining your own class in `APPPATH/classes/Cookie.php` to add your own methods.

[!!] You should **never** modify any of the files that are distributed with Koseven. Always make modifications to classes using transparent extension to prevent upgrade issues.

For instance, if you wanted to create method that sets encrypted cookies using the [Encrypt] class, you would create a file at `APPPATH/classes/Cookie.php` that extends KO7_Cookie, and adds your functions:

    <?php

    class Cookie extends KO7_Cookie {

        /**
         * @var  mixed  default encryption instance
         */
        public static $encryption = 'default';

        /**
         * Sets an encrypted cookie.
         *
         * @uses  Cookie::set
         * @uses  Encrypt::encode
         */
         public static function encrypt($name, $value, $expiration = NULL)
         {
             $value = Encrypt::instance(Cookie::$encrpytion)->encode((string) $value);

             parent::set($name, $value, $expiration);
         }

         /**
          * Gets an encrypted cookie.
          *
          * @uses  Cookie::get
          * @uses  Encrypt::decode
          */
          public static function decrypt($name, $default = NULL)
          {
              if ($value = parent::get($name, NULL))
              {
                  $value = Encrypt::instance(Cookie::$encryption)->decode($value);
              }

              return isset($value) ? $value : $default;
          }

    } // End Cookie

Now calling `Cookie::encrypt('secret', $data)` will create an encrypted cookie which we can decrypt with `$data = Cookie::decrypt('secret')`.

## How it works

To understand how this works, let's look at what happens normally.  When you use the Cookie class, [KO7::autoload] looks for `classes/Cookie.php` in the [cascading filesystem](files).  It looks in `application`, then each module, then `system`. The file is found in `system` and is included.  Of course, `system/classes/Cookie.php` is just an empty class which extends `KO7_Cookie`.  Again, [KO7::autoload] is called this time looking for `classes/KO7/Cookie.php` which it finds in `system`.

When you add your transparently extended cookie class at `application/classes/Cookie.php` this file essentially "replaces" the file at `system/classes/Cookie.php` without actually touching it.  This happens because this time when we use the Cookie class [KO7::autoload] looks for `classes/Cookie.php` and finds the file in `application` and includes that one, instead of the one in system.

## Example: changing [Cookie] settings

If you are using the [Cookie](cookies) class, and want to change a setting, you should do so using transparent extension, rather than editing the file in the system folder.  If you edit it directly, and in the future you upgrade your Koseven version by replacing the system folder, your changes will be reverted and your cookies will probably be invalid.  Instead, create a Cookie.php file either in `application/classes/Cookie.php` or a module (`MODPATH/<modulename>/classes/Cookie.php`).

	class Cookie extends KO7_Cookie {
	
		// Set a new salt
		public $salt = "some new better random salt phrase";
		
		// Don't allow javascript access to cookies
		public $httponly = TRUE;
		
	}


## Multiple Levels of Extension

If you are extending a Koseven class in a module, you should maintain transparent extensions. In other words, do not include any variables or function in the "base" class (eg. Cookie). Instead make your own namespaced class, and have the "base" class extend that one. With our Encrypted cookie example we can create `MODPATH/mymod/Encrypted/Cookie.php`:

	class Encrypted_Cookie extends Kohana_Cookie {

		// Use the same encrypt() and decrypt() methods as above

	}

And create `MODPATH/mymod/Cookie.php`:

	class Cookie extends Encrypted_Cookie {}

This will still allow users to add their own extension to [Cookie] while leaving your extensions intact. To do that they would make a cookie class that extends `Encrypted_Cookie` (rather than `KO7_Cookie`) in their application folder.
