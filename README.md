# BitClout Messages

BitClout Messages is a GUI that allows you to encrypt messages using someone's BitClout address and decrypt messages you receive using your seed phrase. You can send the messages through any medium - like email, twitter message, signal or telegram. 

# How To Create A Message
1. Visit https://freetrade.github.io/BitCloutMessages/
2. Enter the address(public key) of the BitClout reciptient. (Hint, it starts with 'BC1...')
3. Enter the secret message
4. Press 'Encrypt' button
5. Send the generated encrypted message by email or some other method.

# How To Decrypt A Message
1. Visit https://freetrade.github.io/BitCloutMessages/
2. Enter your seed phrase (Hint, it should 12 words, make sure all lower-case and no extra spaces) 
3. Enter the message you received
4. Press the 'Decrypt' button.

# How Do I Do All This Offline?
1. Download the GUI from https://github.com/FreeTrade/BitCloutMessages/archive/refs/heads/main.zip
2. Unzip it to your desktop
3. Go offline.
4. Click on the 'index.html' file to open it
5. Follow Steps above

# How Does All The Code Work?
The code uses two libraries.

Bitbox SDK for converting seed phrase to a private key
https://github.com/Bitcoin-com/bitbox-sdk

EccryptJS library for encrypting and decrypting
https://github.com/pedrouid/eccrypto-js

There are two functions in the index.html file that tie everything together - encrypt, and decrypt here's how they work -

```
async function encryptMessage(bcpubkey, message, element) {

      // Encrypt the message
      try {
        //Decode the BitClout Public Key From BS58 check format
        var preslice = window.bs58check.decode(bcpubkey);

        //Remove the first three characters (these aren't part of the public key and just identify as a BitClout address)
        var bcpublicKey = preslice.slice(3);

        //Convert the public key into a Buffer for use with the encryption library
        const pubKeyBuf = Buffer.from(bcpublicKey, 'hex');

        //Convert the message into a buffer for use with the library
        const data = Buffer.from(message);

        //Encrypt the message using the public key using the eccryptoJS library
        const structuredEj = await eccryptoJs.encrypt(pubKeyBuf, data);

        //Convert the message to a hex string
        const encryptedMessage = eccryptoJs.serialize(structuredEj).toString('hex');

        //Place the hex string in the GUI element.
        element.value = encryptedMessage;
      } catch (err) {
        alert(err);
      }
    }

    async function decryptMessage(loginkey, message, element) {

      // Decrypt the message
      try {

        //User the Bitbox library to get the private key from Seed Phrase
        //uses the standard Bitcoin derivation
        var pkWIF = new bitboxSdk.Mnemonic().toKeypairs(loginkey, 1, false, "44'/0'/0'/0/")[0].privateKeyWIF;

        //Create the cyptographic pair from the privatekeyWIF
        let ecpair = new bitboxSdk.ECPair().fromWIF(pkWIF);

        //the private key as hex
        var privkeyhex = ecpair.d.toHex();

        //Convert private key to buffer for use with crypto library
        var privateKeyBuf = Buffer.from(privkeyhex, 'hex');

        //Convert hex to binary buffer
        const encrypted = eccryptoJs.deserialize(Buffer.from(message, 'hex'));

        //Decrypt the encrypted message using the private key
        const structuredEj = await eccryptoJs.decrypt(privateKeyBuf, encrypted);

        //Place the hex string in the GUI element.
        element.value = structuredEj.toString();

      } catch (err) {
        alert(err);
      }
    }

```






