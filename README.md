# layer moat-encrypt and castle-encrypt to encrypt with aes256 with different libraries multiple times

The draconian-one-way includes two scripts, one to use openssl aes256 to encrypt, the other to use gpg (aes256, or what is otherwise configured in the gpg conf). If you have both libraries and want that extra level of protection, and have the compute time to do so, use both.



/usr/local/sbin/castle-encrypt /home/thane/logs.$(date +%Y%m%d).tgz /home/thane/logs.$(date +%Y%m%d).tgz.asc

(To use moat-encrypt, you will need to have a public pgp key in the user keyring, and then use the key identifier such as email with the script for the user that has the public key imported into their keyring.)

/usr/local/sbin/moat-encrypt /home/thane/logs.$(date +%Y%m%d).tgz.asc adminbobperson@something.place.example.com

# and if you really want to keep going, do that all again with another set of keys!

/usr/local/sbin/castle-encrypt /home/thane/logs.$(date +%Y%m%d).tgz.asc.gpg /home/thane/logs.$(date +%Y%m%d).tgz.asc.gpg.asc

/usr/local/sbin/moat-encrypt /home/thane/logs.$(date +%Y%m%d).tgz.asc.gpg.asc root-example@hsm-thingy-stor-example


After you complete your encryptions, you might move your encrypted tbin file off the server that stores the data to another safe location, shredding the copy of those as well:


  scp 424083f13cd57913bce33fe7bb3aad71.key.bin.enc user@safeserver:/var/storage/key-files/01/
  
  shred -v -n 25 -u -z 424083f13cd57913bce33fe7bb3aad71.key.bin.enc


# about the scripts

# draconian-one-way: castle-encrypt
If for some reason a machine can't use PGP/GNUPG, but can use openssl/libressl, then at least you can use castle-encrypt.

Also see https://github.com/jpegleg/metarc for more related functions and usage info.

This software does not contain any encryption software, just ways to automate or simplify other cryptographic software that is not included here but referenced. Examples: openssl, libressl, gpg, pgp...

This castle-encrypt concept might be used for some ancient UNIX or odd build that conflicts with or can't run GNUPG, and doesn't/can't use PGP5 etc.
You will need (another system, token, person) that has the private RSA key that generated the public key used by the castle-encrypt script, otherwise all of the data will be lost.

Another reason why you might go down this path is that you want to add layers of encryption. You might want to do this,
then encrypt files already encrypted by a process like this with gpg, such as with moat-encrypt script. The idea there is that both cryptosystems have be thwarted
rather than just one, say if you have a powerful and/or patient adversary or adversaries. Most of the time just encrypting with AES256 is all you need, but if your enemy has the time and the compute power and there is a need to escalate the security, stack up on encryption layers. When you have to make several layers, openssl/libressl can be a nice addition to the mix along with gpg etc.

Here is an example wrapper that takes the castle-encrypt script encrypted output and ships it off:

/usr/local/sbin/castle-encrypt /home/thane/logs.$(date +%Y%m%d).tgz /home/thane/logs.$(date +%Y%m%d).tgz.asc

mv /root/*.key.bin.enc /mnt/stor/

mv /home/thane/logs.$(date +%Y%m%d).tgz.asc /mnt/stor/

find /mnt/stor/ -mtime +1 -exec rm -f {} \;


Then on a remote system that has the private RSA key, or yet another server entirely, you might collect the data from 

/mnt/store/

and then gpg it up once per whenever you sync up with the other system. Like have the castle-encrypt run every day at 5:24 AM, and then this next system collecting them at 5:30 AM on Sundays.

rsync -raz --stats --progress thatotherserver://mnt/stor/* /pull/stor/

tar czvf /var/stor/data/store.tgz /pull/stor/*

gpg -r root@example --yes --batch --armor --encrypt /var/store/data/store.tgz

shred -v -n 25 -u -z /pull/stor/*

shred -v -n 25 -u -z /var/store/data/store.tgz

Perhaps the private gpg key is on a yubikey and the private rsa key is on a non-networked physically secure workstation that you have to take the yubikey with the private gpg key, and a "secure" USB drive or drive mount with the twice encrypted data, to decrypt all the way if you need to check on something or otherwise analyze the data. Copy the twice encrypted data from the USB drive to the secure workstation, plug in the yubikey to decrypt the tarball, then run (another program joining this repository soon) that decrypts the tarballs encrypted by the key.bin in castle-encrypt openssl/libressl, in a batch and/or by picking specific type frames based on data file name. Then of course cleaning up after yourself and securely removing the data when you are done with the analysis etc :)


# draconian-one-way: moat-encrypt
Add a layer of GPG encryption with a public PGP key.


You will need to have generated a gpg (pgp) key prior to running this script.
Generate the private gpg key somewhere else safe, or on a liveboot image.
Then bring over the public key to a file called pub.asc
Import pub.asc into user keyring and then call the moat-encrypt script like this:

/usr/local/sbin/moat-encrypt somefile.tbin adminbobperson@something.place.example.com
