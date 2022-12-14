# An extended call routing for AVM FRITZ!Box

<img align="right" src="assets/washing.png"/>

The program is **trying to identify spam calls**. So it is listen to the [FRITZ!Box](#disclaimer) callmonitor and does several wash cycles to figure out whether it is spam or not.

## Release notes

If you already use an older version please refer to the [update section](#update) and be aware that the **structure of the configuration file differ** (see [`config.example.php`](/config.example.php)).

### Brand new (v2.8)

Unknown outgoing phone numbers are now also subjected to a reverse search via [Das Örtliche](https://www.dasoertliche.de/rueckwaertssuche/) and, if necessary, recorded in the corresponding phone book.

### Recently added (v2.7)

At the request of a user, an **e-mail notification** has been added. If an unknown number has led to an entry in one of the phone books, you can get logging information as an email. Could the number be researched on the web (see next paragraph) - as a spammer or by reverse search - even with the info where it was found as a **deep link**: so you can dive into detailed information to that phone number direct from the e-mail.

## Preconditions

* the program only works in the German telephone network!
* you use a Fritz!Box for landline telephoning
* you need to have a [separate spam telefon](https://avm.de/service/wissensdatenbank/dok/FRITZ-Box-7590/142_Rufsperren-fur-ankommende-und-ausgehende-Anrufe-z-B-0900-Nummern-einrichten/) book beside your phone book!
* you have a microcomputer (Raspberry Pi) running 24/7 in your network

## Description

For an incoming call a cascaded check takes place:

* First, it is checked, whether the number is **already known** in your telephone books (`'whitelist'`) and (`'blacklist'`) or (`'newlist'`). Of course, these known telephone numbers are not analyzed further.

* If unknown, than it is checked if it is a **foreign number**. If you have set (`'blockForeign'`) the number will be direct transferred to the corresponding phone book (`'blacklist'`) for future rejections.

* Than it is checked if a domestic number has a [**valid area code** ONB](#onb) or [cellular code](#rnb). Quite often spammers using fake area codes. If so, the number will be transferred to the blacklist for future rejections.

* After that it is checked if the **subscribers number is valid** (starting with a zero -> only applies to landline numbers). If so, the number will be transferred to the blacklist.

* If all this passed, it is checked at various scoring sites (currently five: [WERRUFT](https://www.werruft.info), [Clever Dialer](https://www.cleverdialer.de), [Telefonspion](https://www.telefonspion.de/), [WerHatAngerufen](https://www.werhatangerufen.com) and  [tellows](https://www.tellows.de/)), if this number has received a **bad online rating**. If so, the number will be transferred to the blacklist.

* Finally, of course, there is the possibility that the caller is known in a positive sense and can be identified via a public telephone book ([Das Örtliche](https://www.dasoertliche.de/rueckwaertssuche/)). Then he/she/it is optionally entered in a dedicated phone book (`'newlist'`) with the determined name.

## Requirements

* PHP >= 7.0 (php-cli, php-curl, php-mbstring, php-soap, php-xml)
* callmonitor (port 1012) is open - if not: dial `#96*5*` to open it
* Composer (follow the installation guide at <https://getcomposer.org/download/)>

## Installation

### Programm

Install requirements are:

```console
git clone https://github.com/blacksenator/fbcallrouter.git
cd fbcallrouter
```

Install composer (see <https://getcomposer.org/download/> for newer instructions):

```console
composer install --no-dev --no-suggest
```

[Edit](#configuration) the `config.example.php` and save as `config.php` or use an other name of your choice (but than keep in mind to use the -c option to define your renamed file)

### Preconditions on the FRITZ!Box

#### Phone book for rejections

If you do not have your own phone book for spam numbers, it is essential to add a new one (e.g. "Spamnummern"). Note that the first phone book ("Telefonbuch") has the index "0" and the numbers are ascending according to the index tabs.
Then you have to [link this phone book for call handling](https://avm.de/service/wissensdatenbank/dok/FRITZ-Box-7590/142_Rufsperren-fur-ankommende-und-ausgehende-Anrufe-z-B-0900-Nummern-einrichten/).

#### FRITZ!Box user

The programm accessed the Fritz!Box via TR-064. Make sure that the user you choose in configuration (see [next topic](#configuration)) is granted for this interface. The proposed user `dslf_config` is the default user when logging in with a password and without user selection and has the required rights!

### Configuration

The configuration file (default `config.php`) offers various customization options. For example: Adapted to the tellows rating model a bad score is determined as six to nine and needs at least more than three comments. The second parameter is for quality purposes: not a single opinion should block a service provider whose call you might expect.
You can adapt the values in the configuration file. **But be carefull! If you choose score values equal or smaler then five (5), than numbers with impeccable reputation where written to your rejection list!**
Because the rating websites use different scales, they are normalized to the tellows scale.

If you set `'log'` in your configuration and the `'logPath'` is valid, the essential process steps for verification are written to the log file `callrouter_logging.txt`. If `'logPath'` is empty the programm directory is default.

## Usage

As mentioned, the program is designed to run permanently in the background. In order to ensure this, it is necessary to make sure that this is possible without interruption and should be tested accordingly.

### Test

It is highly recommended test the setting by following the next steps:

#### 1. Function test

```console
php fbcallrouter run
```

If `On guard...` appears, the program is armed.

Make a function test in which you call your landline from your cell phone: your mobile phone number **should not** end up in the spam phone book!
If the number is in your phone book, nothing should happen anyway (on whitelist).
Otherwise: your mobile phone number is not a foreign number, the [ONB](#onb)/[RNB](#rnb) is correct and you certainly do not have a bad entry in online directories. Therefore these tests should not lead to any sorting out.
Press `CTRL+C` to terminate the programm.

If logging is enabled (which is highly recommended for this test), than `nano callrouter_logging.txt` will show you what happend at the callmonitor interface.

#### 2. Integration test

There are five exemplary `'numbers'` stored in the configuration file with which you can test the wash cycles integratively. You can change these test numbers according to your own ideas and quantity. Starting the programm with the `-t` option, these numbers will be injected as substitutes one after the other for the next calls the FRITZ!Box receives and its callmonitor port will broadcast.

It is highly recommended to proceed like this:

1. check if none of the substitutes are already in your phone book! **Especially if you repeat the test!**

2. if you want to a web query, make crosscheck at the named providers that the choosen test phone number(s) actually **has/ve the desired score and comments!**

3. start the programm

    ```console
    php fbcallrouter run -t
    ```

4. use your cellular phone to call your landline. The incoming mobil number will be replaced with the first/next number from this array and passes through the inspection process. Additional information is output like this:

    ```console
    Starting FRITZ!Box call router...
    On guard...
    Running test case 1/5
    ```

5. hang up and repeat calling from your cellular your landline number. So you have to call as many times as there are numbers in the array to check all test cases (or quit programm execution with `CTRL+C`). The program then ends this number replacement.

6. check the blacklist phone book whether all numbers have been entered as expected. If logging is enabled, than `nano callrouter_logging.txt` will show you what happend. If e-mail notification is choosen check your inbox.
To cancel, press `CTRL+C`.

### 3. Permanent background processing

The **main concept** of this tool is to **continuously check incoming calls**. Therefore it should ideally run permanently as a background process on a single-board computer (e.g. Raspberry Pi) in the home network. A corresponding definition file is prepared: [`fbcallrouter.service`](/fbcallrouter.service)

1. edit `[youruser]` in this file with your device user (e.g. `pi`) and save the file

    ```console
    nano fbcallrouter.service
    ```

2. copy the file into `/etc/systemd/system`

    ```console
    sudo cp fbcallrouter.service /etc/systemd/system/fbcallrouter.service
    ```

3. enable the service unit:

    ```console
    sudo systemctl enable fbcallrouter.service
    ```

4. check the status:

    ```console
    sudo systemctl status fbcallrouter.service
    ```

## Update

As noted at the beginning, functional enhancements usually go hand in hand with additions to the configuration file. To install the current version, please proceed as follows:

1. change to the installation directory:

    ```console
    cd /home/[youruser]/fbcallrouter
    ```

2. stop the service:

    ```console
    sudo systemctl stop fbcallrouter.service
    ```

3. delete the old logging file (if you used it here):

    ```console
    rm callrouter_logging.txt
    ```

4. get the latest version from:

    ```console
    git pull https://github.com/blacksenator/fbcallrouter.git
    ```

5. bring all used libraries up to date:

    ```console
    composer update --no-dev
    ```

6. check for changes in the configuration file...

    ```console
    nano config.example.php
    ```

7. ...and eventually make necessary changes/additions in your configuration:

    ```console
    nano config.example.php
    ```

8. restart the service...

    ```console
    sudo systemctl start fbcallrouter.service
    ```

9. ...and wait few seconds before you check if the service is running:

    ```console
    sudo systemctl status fbcallrouter.service
    ```

### Master data

#### ONB

ONB = OrtsNetzBereiche (Vorwahlbereiche/Vorwahlen). The list used comes from the [BNetzA](https://www.bundesnetzagentur.de/DE/Fachthemen/Telekommunikation/Nummerierung/ONRufnr/ortsnetze_node.html) and should be valid for a limited period of time. If you want to update them, then download the offered **CSV file** [Vorwahlverzeichnis](https://www.bundesnetzagentur.de/SharedDocs/Downloads/DE/Sachgebiete/Telekommunikation/Unternehmen_Institutionen/Nummerierung/Rufnummern/ONRufnr/Vorwahlverzeichnis_ONB.zip.zip?__blob=publicationFile&v=298). Unpack the archive and save the file as `ONB.csv` in the `./assets` directory.

#### RNB

RNB = numbers for mobile services. The BNetzA do not provide a list for download with cellular codes [(RNB)](https://www.bundesnetzagentur.de/DE/Fachthemen/Telekommunikation/Nummerierung/MobileDienste/start.html). The currently used ones were transferred to the `./assets` directory as `$cellular Numbers` in `cellular.csv`.

## Does the programm works propperly?

First of all: of course the program can contain bugs and not work as expected. If you are convinced, please open an issue here.
In addition, it can of course be more due to the selected settings for the configuration of the `filter`.

Last but not least I myself have made some observations that led me to suspect that the program would not work correctly. In principle, it is advisable in such cases to **switch on logging** (`'log' => true,`) or to compare the active logging with the call list of the FRITZ!Box. You may need to go a step further and correlate dates with reverse search sites.

An example:
I had calls that should have been identified as spam at first glance through web research. A closer look showed that this spammer only appeared in the lists at exactly that time and had not yet received a sufficient number of negative comments at the time of the call.

## Privacy

No private data from this software will be passed on to third parties accepts with this exception:
when using

* [WERRUFT](https://www.werruft.info/bedingungen/),
* [Clever Dialer](https://www.cleverdialer.de/datenschutzerklaerung-website)
* [Telefonspion](https://www.telefonspion.de/datenschutz.php)
* [WerHatAngerufen](https://www.werhatangerufen.com/terms)
* [tellows](https://www.tellows.de/c/about-tellows-de/datenschutz/)
* [Das Örtliche](https://www.dasoertliche.de/datenschutz)

the incoming number is transmitted to the provider. Their data protection information must be observed!

## Disclaimer

FRITZ!Box, FRITZ!fon, FRITZ!OS are trademarks of [AVM](https://avm.de/). This software is in no way affiliated with AVM and only uses the [interfaces published by them](https://avm.de/service/schnittstellen/).

## Improvements

As ever this program started with only a few lines of code just to play with the callmonitor interface provided and to figure out how it works...

...than more and more ideas came to my mind how this interface could solve some of my needs.

If you have an idea for a useful improvement or addition, then please open an issue.

As I have already written in the [fritzsoap documentation](https://github.com/blacksenator/fritzsoap#wishes), it would be an enormous relief **if AVM would provide functionality to terminate incoming calls**, just as FRITZ!OS itself does it with the handling of phone numbers to be blocked. The entry of more and more numbers in corresponding blacklists could be omitted or reduced.

## License

This script is released under MIT license.

## Author

Copyright© 2019 - 2022 Volker Püschel
