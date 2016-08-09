# Secrets-Cipher-Base64 

[![Gem Version](https://badge.fury.io/rb/secrets-cipher-base64.svg)](https://badge.fury.io/rb/secrets-cipher-base64)
[![Downloads](http://ruby-gem-downloads-badge.herokuapp.com/secrets-cipher-base64?type=total)](https://rubygems.org/gems/secrets-cipher-base64)

<br />

[![Build Status](https://travis-ci.org/kigster/secrets-cipher-base64.svg?branch=master)](https://travis-ci.org/kigster/secrets-cipher-base64)
[![Code Climate](https://codeclimate.com/github/kigster/secrets-cipher-base64/badges/gpa.svg)](https://codeclimate.com/github/kigster/secrets-cipher-base64)
[![Test Coverage](https://codeclimate.com/github/kigster/secrets-cipher-base64/badges/coverage.svg)](https://codeclimate.com/github/kigster/secrets-cipher-base64/coverage)
[![Issue Count](https://codeclimate.com/github/kigster/secrets-cipher-base64/badges/issue_count.svg)](https://codeclimate.com/github/kigster/secrets-cipher-base64)

## Summary

What? *Another security gem?* —— Well, funny you should ask! 

You see, security is an incredibly wide topic. The tools around security tend to fall into two classes: swiss army knife wrappers around `OpenSSL`, and more specialized tools. This gem falls in the second category. It wraps a very small part of `OpenSSL` library. 

> __Namely, `secrets-cipher-base64` provides:__
>
> * Symmetric data encryption with: 
>   * the cipher `AES-256-CBC` 
>   * 256-bit private key
>   * which can be optionally password-encrypted.
> * Rich command line interface with some innovative features, such as inline encrypted file editing using your current `$EDITOR`
> * Automatic compression of the data
> * Rich Ruby API and highly extensible approach to encryption/decryption
> * Automatic detection of password-protected keys, 
> * and more...

The main point behind this gem is to allow you to store sensitive application secrets in your source code repo as `AES-256-CBC`-encrypted files or strings (this is the same encryption algorithm that US Government uses internally). The output of the encryption is always a (urlsafe) `base64`-encoded string, without the linebreaks.
 
The private key (encrypted or not) is also a base64-encoded string, typically 45 characters long (unless it's password encrypted, in which case it is considerably longer). 
 
Using a single-line string that `urlsafe_encode64()` generates, makes this gem well suited for encrypting specific fields in the YAML file, or simply the entire file.  This also means that the private key can be easily exported as an environment variable (as it's value is a single-line string).  
 
> NOTE: The library leverages the code shown in the following discussion: http://stuff-things.net/2015/02/12/symmetric-encryption-with-ruby-and-rails/ I'd like to acknowledge the the author of the above thread for providing clear examples of a simple symmetric encryption with `OpenSSL`.

## Symmetric Encryption

Symmetric encryption simply means that we are using the same private key to encrypt and decrypt. The secret can be generated by the tool and is a *base64-encoded* string which is 45 characters long. The *decoded* secret is always 32 characters long (or 256 bytes long).

In addition to the private key, the encryption uses an IV vector. The library completely hides `iv` from the user, generates one random `iv` per encryption, and stores it together with the field itself (*base64-encoded*).

## Installation

If you plan on using the library in your ruby project with Bundler managing its dependencies, just include the following line in your `Gemfile`:

```ruby
gem 'secrets-cipher-base64'
```
And then run `bundle`.

Or install it into the global namespace with `gem install` command:

    $ gem install secrets-cipher-base64
    $ secrets -h
    $ secrets -E # see examples

## Usage

### Private Keys

This library relies on the existance of the 32-byte private key (aka, *a secret*), that must be stored somewhere safely if your encrypted data is to be persisted, for example it can be saved into the keychain on Mac OSX. 

> In fact, we put together a separate file that discusses strategies for protecting your encryption keys, for example you can read about [how to use Mac OS-X Keychain Access application](https://github.com/kigster/secrets-cipher-base64/blob/master/MANAGING-KEYS.md) and other methods. Additions and discussion are welcome. Please contribute!

You can use one key for all encrypted fields, or many keys – perhaps one per deployment environment, etc. While you can have per-field secrets, it seems like an overkill.

__NOTE: it is considered a bad practice to check in the private key into the version control.__  If you keep your secret out of your repo, you can check-in encrypted secrets file directly into the repo. As long as the private key itself is safe, the data in your encrypted  will be next to impossible to extract. 

### Command Line (CLI)
 
You can generate using the command line, or in a programmatic way. First we'll discuss the command line usage, and in a later section we'll discuss Ruby API provided by the gem. 

#### Generating and Using Private Keys

Once the gem is installed you will be able to run an executable `secrets`:

```bash
gem install secrets-cipher-base64

# then let's generate and copy the new private key to the clipboard.
# Clipboard is activated with the -c flag does.
secrets -gc

# or save a new key into a bash variable
SECRET=$(secrets -g)

# or create a password-protected key, and save it to a file:
secrets -gcp -o ~/.secret
# New Password:     ••••••••••
# Confirm Password: •••••••••• 
```

You can subsequently use the private key by either:

  1. passing the `-k [key value]` flag 
  2. passing the `-K [filename]` flag

NOTE: If you installed the gem with bundler, make sure to prefix the above commands with `bundle exec`).

####  Encryption and Decryption

This may be a good time to take a look at the full help message for the `secrets` tool:

```bash
❯ exe/secrets -h
Usage:
    secrets [options]
Modes:
    -g, --generate                generate a new private key
    -e, --encrypt                 encrypt
    -d, --decrypt                 decrypt
    -t, --edit                    decrypt and edit a file in vim
Options:
    -p, --password                encrypt key with a password
    -k, --private-key  [key]      file containing private key
    -K, --key-file     [file]     specify the encryption key
    -s, --string       [string]   specify a string to encrypt/decrypt
    -f, --file         [file]     filename to read from
    -o, --output       [file]     filename to write to
    -i, --interactive             ask for a key interactively
    -b, --backup                  create a backup file in the edit mode
    -c, --copy                    when used with -g copies the key to clipboard
Flags:
    -v, --verbose                 show additional information
    -T, --trace                   print a backtrace of any errors
    -E, --examples                show several examples
    -V, --version                 print library version
    -N, --no-color                disable color output
    -h, --help                    show help```

```

#### Examples of CLI Usage

__Generating the Key__:

```bash
# generate a new private key into an environment variable:
export KEY=$(secrets -g)
echo $KEY
75ngenJpB6zL47/8Wo7Ne6JN1pnOsqNEcIqblItpfg4=
# ————————————————————————————————————————————————————————————————————————————————
# generate a new password-protected key, copy to the clipboard & save to a file
secrets -gpc -o ~/.key
New Password     : ••••••••••
Confirm Password : ••••••••••
# ————————————————————————————————————————————————————————————————————————————————
# encrypt a plain text string with a key, and save the output to a file
secrets -e -s "secret string" -k $KEY -o file.enc
cat file.enc
# => Y09MNDUyczU1S0UvelgrLzV0RTYxZz09CkBDMEw4Q0R0TmpnTm9md1QwNUNy%T013PT0K
# ————————————————————————————————————————————————————————————————————————————————
# decrypt a previously encrypted string:
secrets -d -s $(cat file.enc) -k $KEY
secret string
# ————————————————————————————————————————————————————————————————————————————————
# encrypt secrets.yml and save it to secrets.enc:
secrets -e -f secrets.yml -o secrets.enc -k $KEY
# ————————————————————————————————————————————————————————————————————————————————
# decrypt an encrypted file and print it to STDOUT:
secrets -df secrets.enc -k $KEY
# ————————————————————————————————————————————————————————————————————————————————
# edit an encrypted file in $EDITOR, ask for key, create a backup
secrets -tibf ecrets.enc
# => Private Key: ••••••••••••••••••••••••••••••••••••••••••••
# => Saved encrypted content to secrets.enc.

# => Diff:
3c3
# # (c) 2015 Konstantin Gredeskoul.  All rights reserved.
# ---
# # (c) 2016 Konstantin Gredeskoul.  All rights reserved.
# ————————————————————————————————————————————————————————————————————————————————
```

##### Inline Editing

The `secrets` CLI tool supports one interesting mode where you can open an encrypted file in an `$EDITOR`, and edit it's unencrypted version (stored temporarily in a temp file), and upon saving and exiting the gem will automatically diff the new and old content, and if different – will save encrypt it and overwrite the original file.

In this mode several flags are of importance:

    -b (--backup)   – will create a backup of the original file
    -v (--verbose) - will show additional info about file sizes
      
Here is a full command that opens a file specified by `-f | --file`, using the key specified in `-K | --key-file`, in the editor defined by the `$EDITOR` environment variable (or if not set – defaults to `/bin/vi`)". 

NOTE: while much effort has been made to ensure that the gem is bug free, the reality is that no software is bug free. Please make sure to backup your encrypted file before doing it for the first few times to get familiar with the command.
     
    secrets -tbv -K ~/.key 

### Ruby API

To use this library you must include the main `Secrets` module into your library.

Any class including `Secrets` will be decorated with new class methods `#private_key` and `#create_private_key`, as well as instance methods `#encr`, and `#decr`. 

`#create_private_key` will generate a new key each time it's called, while `#private_key` will either assign an existing key (if a value is passed), or generate and save a new key in the class instance variable. Therefore each class including `Secrets` will use it's own key (unless the key is assigned). 

The following example illustrates this point:

```ruby
require 'secrets'

class TestClass
  include Secrets
end
@key = TestClass.create_private_key
@key.eql?(TestClass.private_key)  # => false
# A new key was created and saved in #private_key accessor.

class SomeClass
  include Secrets
  private_key TestClass.private_key
end

@key.eql?(SomeClass.private_key)  # => true (it was assigned)
```

#### Encryption and Decryption

So how would we use this library from another ruby project to encrypt and decrypt values?

After including the `Secrets` module in a ruby class, the class will now have the `#encr` and `#decr` instance methods, as well as `#secret` and `#create_private_key class methods.

Therefore you could write something like this below, protecting a sensitive string using a class-level secret.

```ruby
require 'secrets'
class TestClass
  include Secrets
  key ENV['SECRET']
  
  def sensitive_value=(value)
    @sensitive_value = encr(value, self.class.private_key)
  end
  def sensitive_value
    decr(@sensitive_value, self.class.private_key)
  end
end
```

### Configuration

The library offers a typical `Secrets::Configuration` class which can be used to tweak some of the internals of the gem. This is really meant for a very advanced user who knows what she is doing. The following snippet is actually part of the Configuration class itself, but can be overridden by your code that uses and initializes this library. `Configuration` is a singleton, so changes to it will propagate to any subsequent calls to the gem.

```ruby
Secrets::Configuration.configure do |config|
  config.password_cipher = 'AES-128-CBC'  # 
  config.data_cipher = 'AES-256-CBC'
  config.private_key_cipher = config.data_cipher
  config.compression_enabled = true
  config.compression_level = Zlib::BEST_COMPRESSION
end
```

As you can see, it's possible to change the default cipher typem, although not all ciphers will be code-compatible with the current algorithm, and may require additional code changes.

## Managing Keys

There is a separate discussion about ways to securely store private keys in [MANAGING-KEYS.md](https://github.com/kigster/secrets-cipher-base64/blob/master/MANAGING-KEYS.md).

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/kigster/secrets-cipher-base64.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

## Author

This library is the work of [Konstantin Gredeskoul](http:/kig.re), &copy; 2016, distributed under the MIT license.

