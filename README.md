# SimpleSAMLphp-QuickStart
The missing quick start guide for setting up SimpleSAMLphp as a SAML IdP

Given that "Simple" is in its name, you'd expect SimpleSAMLphp would be quick and easy to setup as an IdP. Well, it is, but not if you try following its obtuse guide or using the default config files. So, here's a quick way to get started if you need an IdP for testing purposes.

Instead of setting up a whole new server/port, this guide sets up SimpleSAMLphp (SSP) IdP on an existing web server, doing away with having to configure Apache/Nginx. Use care with sessions/etc if web app is being tested in on same server, or if you are using a shared server and giving web server permission to read certificate key.

This guide is based on SSP 2.4.2.

1. Download the software and uncompress it outside the web document root.
```
   % cd /usr/local/src/
   % wget https://github.com/simplesamlphp/simplesamlphp/releases/download/v2.4.2/simplesamlphp-2.4.2-full.tar.gz
   % tar xzf simplesamlphp-2.4.2-full.tar.gz
   % mv simplesamlphp-2.4.2 simplesamlphp
```

3. Symlink the public directory to /simplesaml/ in your web document root.
```
   % cd /var/www/
   % ln -s /usr/local/src/simplesmalphp/public simplesaml
```

4. Create a key/cert for the IdP
```
   % cd simplesaml/cert/
   % openssl req -newkey rsa:3072 -new -x509 -days 3652 -nodes -out example.org.crt -keyout example.org.pem
   % chmod 644 *     (or make it so web server process user can read .crt and .pem)
```

5. Configure SSP
```
   % cd ../config/
   % cp config.php.dist config.php
   % vi (or emacs) config.php
       Change settings as follows:
         'baseurlpath' => 'https://SERVER_NAME/simplesaml/',
         'technicalcontact_email' => 'you@somewhere.rainbow',
         'secretsalt' => '4urh34t8yutrfhn4hg83h48fq',       (or some other nonsense)
         'auth.adminpassword' => 'TopSecret',               (or whatever you want)
         'enable.saml20-idp' => true
         'module.enable' => [
           'exampleauth' => true,
           'admin' => true,
           // comment out any others
         ],
```

6. Create Auth Sources
 ```
  % vi (or emacs) authsources.php
       Set content as follows (feel free to modify):
<?php
$config = [
    'admin' => [ 'core:AdminPassword' ],
    'exampleauth' => [
        'exampleauth:UserPass',
        'users' => [
            'test:test123' => [
                'uid' => ['test'],
                'email' => ['test@example.org'],
                'eduPersonAffiliation' => ['member', 'student'],
            ],
            'employee:employeepass' => [
                'uid' => ['employee'],
                'email' => ['employee@example.org'],
                'eduPersonAffiliation' => ['member', 'employee'],
            ],
        ],
    ],
];
```

7. Setup metadata
```
  % cd ../metadata/
  % cp saml20-idp-hosted.php.dist saml20-idp-hosted.php
  % vi (or emacs) saml20-idp-hosted.php
      Change
        $metadata['urn:x-simplesamlphp:example-idp'] = [
      To
        $metadata['https://SERVER_NAME/simplesaml/idp'] = [
        
      Change 'privatekey' and 'certificate' values to the .crt and .pem names set in step 4

      Change
        'auth' => example-userpass',
      To
        'auth' => 'exampleauth',

      Add
        'entityId' => 'https://SERVER_NAME/simplesaml/idp',
      After
        'host' => '__DEFAULT__',
```

8. Create remote SP configuration
```
   % vi (or emacs) saml20-sp-remote.php
       Set content as follows (modify for your actual SP server/path):
<?php
$metadata['https://SP_SERVER/SAML_PATH/'] = [
    'AssertionConsumerService' => [
        [
            'Location' => 'https://SP_SERVER/SAML_PATH/ACS',
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST',
        ],
    ],
    'SingleLogoutService' => [
        [
            'Location' => 'https://SP_SERVER/SAML_PATH/SLS',
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect',
        ],
    ],
];
```

9. Breathe

At this point you should have a functioning IdP. Suggested next steps:

- Access admin panel to check for any errors (auth.adminpassword set in step 6)
  https://SERVER_NAME/simplesaml/admin/

- Download IdP metadata
  https://SERVER_NAME/simplesaml/saml2/idp/metadata.php

- Load IdP metadata file to your SP

- Test login
