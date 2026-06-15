



# JUNK - leftover phar deserialisation question working-out



This means, when the `$CustomTemplate` object is created, and the `$template_file_path`

, it performs a string concatenation in `lockFilePath()` to resolve its path:

```php
return 'templates/' . $this->template_file_path . '.lock';
```

Because `$this->template_file_path` contains the nested `$Blog` object rather than a standard string, PHP is forced to execute a type-juggling conversion. This automatically invokes the `__toString()` magic method inside our injected `Blog` object.

Once `__toString()` fires, it natively triggers the Twig template engine's compilation process:

```php
return $this->twig->render('index', ['user' => $this->user]);
```

As the malicious PHAR file has been successfully uploaded and processed by the system wrapper, the application dynamically builds the template using our malicious `$desc` string payload. The Twig 1.19 environment processes the sandbox escape callback, rendering the exploit completely successful and executing our arbitrary shell commands directly on the host server.





This means, when `$CustomTemplate` object is created, it performs `file_exists($this->lockFilePath())`, which is a **filesystem operation on a user-controllable value.**
- file_exists("`phar://)
- 
- and then setting *that* object to and* as the file was successfully uploaded & contains 


`CustomTemplate`
`New Blog(desc=malicious-twig-SSTI)`
This acts as our **entry point (Sink):** aka, contains a filesystem function (i.e. `file_exists()`, `file_get_contents()`) that accepts a **user-controlled path and supports the stream wrapper** (`phar://`), which is what triggers the implicit `unserialize()` of the PHAR metadata.

CustomTemplate.php uses the `$template_file_path` variable to perform various actions on the file (creating a lockfile, checking if one already exists,)

**`GET /cgi-bin/CustomTemplate.php~`:**
``` php
<?php

class CustomTemplate {
    private $template_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
    }

    private function isTemplateLocked() {
        return file_exists($this->lockFilePath());
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lockFilePath(), "") === false) {
                throw new Exception("Could not write to " . $this->lockFilePath());
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        @unlink($this->lockFilePath());
    }

    private function lockFilePath()
    {
        return 'templates/' . $this->template_file_path . '.lock';
    }
}

?>
```

Reading the code, we need to identify any gadget chains leading to magic methods, performed on the profile picture/`phar`.  

The code can be instantiated if a `phar` is uploaded as a profile picture, as when the picture has finished processing 

So, we need:
- **A method** that causes our malicious php code to be deserialized from the `.phar` metadata (i.e. `file_exists()`, `file_get_contents()`)
	- The one we want is `isTemplateLocked()`, as that sets `lockFilePath()`, which we can control as an attacker via the 
- **An entry point (Sink):** A filesystem function (i.e. `file_exists()`, `file_get_contents()`) that accepts a **user-controlled path and supports stream wrapper**, being what triggers the implicit `unserialize()` of the PHAR metadata.
- 


So, when an avatar is uploaded, a few things happen:
- File is sent to server
- During file processing ()
- `?avata=wiener`
![](attachments/Pasted%20image%2020260610184910.png)