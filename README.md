# DIY GSM Cell Tower Using Yate and Yate-BTS 
By: Sean Walker, Abby Conoyer, Cristin Durbin
```
This guide explains the intricacies involved in establishing a personal cell tower. Using Ubuntu, Open Source Applications, and some specialized hardware,  we explain how one can go about connecting Unlocked-GSM phones to a personal tower. Hence, alleviating the need for a cellular service provider.
```
**CAUTION: This project was strictly performed in an educational environment. Pursuing this project as a means of malintent is strictly prohibited by federal law. If one chooses to channel promiscuous mode, they choose to do so at their own risk. Consider this a warning. We will not be held responsible.
```
Disclaimer: Continue with this tutorial if and only if the intended use is confined to an institutional domain. The legal consequences should also be considered. 
```
### Materials needed for this project:

BladeRF x40 ($420)
Host running ubuntu 16.4 32-bit
2 GSM-compatible phones
Programmable Sim Cards (around $10)

### Software 
Please begin by downloading Ubuntu 16.4 on a flashdrive. It is crucial that the image grabbed is compatible with the host environment. Since we are using a 32 bit laptop, we installed the 32 bit operating system. However, if your machine is 64 bit, opt to place the 64 bit image on the flash drive instead.

After verifying that the appropriate image resides on the bootable USB, or flash drive, connect the USB to one of the host’s many peripheral ports. Then move the Ubuntu operating system from the USB device to the local host. Follow the prompts provided by the install-assistant. Once the configurations have been made and Ubuntu has been stood up on the host, safely eject the USB drive from the laptop. 
```
Now that Ubuntu has been installed on the host, open the terminal. We will begin installing the dependencies needed by our tower. In order to install the first dependency, type the following command from the terminal: 

sudo apt-get install libusb-1.0-0-dev 
sudo apt-get install git 
sudo apt-get install autoconf
sudo apt-get install bladerf
```
Before anything else is done, it is crucial that you create a directory which will store the many source files. Our directory resides on the Desktop. In order to follow along we encourage you to do the same. This directory will serve as repository ensuring that our open source tools can call upon the needed source files. Creating a directory may be done in the GUI or via the terminal. Name the folder whatever you would like. Since we will be referencing the path of this directory rather frequently, choose something that you can easily recall. We recommend naming it ‘Project’-or similar.

After you establish a directory on the desktop, use the terminal, or the gui, to access the newly created directory. In order to access the directory via the terminal use the ls and cd commands. 

While in the Desktop directory, type ls into the terminal and press enter. This depicts the files residing in Desktop. Here you should see the directory you just created. Now use the cd command to transverse into that directory.
	cd NAME_OF_NEW_DIRECTORY {enter}

While in this new directory, let's call it Project, download the FPGA and the firmware files which support the bladeRF. In order to extract these files, type the following commands and press enter. **NOTE-The // signify comments these are not to be included in the commands. These are there for reference only**

wget https://www.nuand.com/fpga/v0.1.2/hostedx40.rbf //BladeRF FPGA 0.1.2
wget https://www.nuand.com/fx3/bladeRF_fw_v1.6.1.img //BladeRF Firmware 1.6.1

```
The next step is to install the yate and yate-bts packages. If you choose to forgo our versions used, understand that you will have to play around with the version numbers of Yate and YateBTS to see which ones are compatible.  With help from the guys over at  https://github.com/security-geeks/evilbts we were able to obtain the latest compatible versions. You can download the two packages here. Or pay a visit to the security geek’s github-https://github.com/security-geeks/evilbts. This file is of type tarball. As a result, we will use the command tar xvzf “filename” in order to extract its contents. Upon doing so, you will see a folder called evilbts. Now use the cd command to examine its contents.

cd evilbts 

Now that you are in the evilbts folder perform an ls command -still using the terminal. You will see the folders denoted above: Yate and YateBTS. First cd into the yate directory and run the following commands:

./autogen.sh 
./configure --prefix=/usr/local
make -j4
sudo make install 
sudo ldconfig 
cd ..

Then cd into the yatebts folder and run the following commands:

./autigen.sh
./configure --prefix=/usr/local 
make -j4 
sudo make install 
sudo ldconfig
cd ..

Either process will take a few minutes to complete. However, once they do, you’ll have everything installed on your system.
```
## Configuration 
```
The  configurations are the most important part of the project. During this section we will configure the BladeRF so that it works with GSM phones. In order to do this we need to modify a file residing in the yate directory. In order to access this file, we will need to follow its path. Perform the following:

	cd /usr/local/etc/yate
Then ls the contents of Yate. You will notice a file named ybts.conf. For us to modify the contents of this file, we will need root privileges. Type: 
	
sudo nano ybts.conf   


Nano is a text editor which allows us to manipulate the contents of ybts.conf. Although vi is another viable text editor, we recommend using nano because we found it easier to use as beginners in Linux. While in the Nano text editor, use the key sequence ‘ctrl’ + ‘w’ to conduct a search of the file’s contents. We found the search function to be handy in modifying the fields of In ybts.conf. The fields in need of entries are those depicted below.

Radio.Band=900
Radio.C0=1000
Identity.MCC=YOUR_COUNTRY_MCC (in the USA is 001)
Identity.MNC=YOUR_OPERATOR_MNC (in the USE is 01)
Identity.shortName=MyEvilBTS
Radio.PowerManger.MaxAttenDB=35
Radio.PowerManger.MinAttenDB=35
```
Save the contents of the text file. Once the options have been saved, close the text editor. If you are looking for values other than those of the United States, visit this link to obtain that MCC and MNC information. 

**NOTE- We couldn’t find the Identity.shortName variable in our ybts.conf file. If it isn’t readily apparent in yours, skip said field. It does not affect the project.**

The next step is to edit the subscriber list. The subscriber list determines which phones are allowed to connect to the BladeRF tower. Use 
cd /usr/local/etc/yate 
which directs you towards the path of the subscriber file. Now that you’re in /usr/local/etc/yate, use the text editor to modify the contents of subscribers.conf. Specifically look for the following section in subscribers.conf:

country_code=YOUR_CONTRY_CODE
regexp=.* 

The regexp will retain the list of authorized devices able to connect. We left this field as a wildcard character depicted above by the asterisk. 

**NOTE: By default the regexp value populates to .* mode. This is what many deem promiscuous mode. Promiscuous mode insinuates that every nearby phone will connect to the tower. Although promiscuous mode is highly entertaining, it is also highly illegal.**

## Start Broadcasting 

If you have made it this far, you are ready to broadcast. In order to start broadcasting go to the command prompt and type:

sudo yate -s

If everything was configured correctly you will see something which resembles the following.

Starting MBTS...
Yate engine is initialized and starting up on Ubuntu
RTNETLINK answers: File exists
MBTS ready  

The program will begin aggregating time stamps. 



**NOTE-If you have trouble broadcasting at this point, look to the BladeRF. Check to see if there are three green dots statically staying on. If this is the case, remove the cables from the Blade hence ceasing power. Then reinsert the cables and run the sudo yate -s  command again **
Testing the BladeRF tower
In the current configuration, the wildcard configuration, any GSM phone will connect to the tower if they are close  enough. You can expect your device to connect within 5-10 minutes. Once granted a connection, you will get a welcome message. Welcoming you to the network, this message identifies your phone number. You will exchange this with the other users on the network to exchange text messages and establish phone calls.


Programing a Sim Card  
Before you can connect to the tower, you must have a compatible SIM card. If your device is locked by your cell phone provider, you must program your own SIM. You will program the SIM using an open source tool called PySIM. Store these files in their own directory. In other words, perform a 

mkdir PySIM

While in the PySIM directory, obtain the program by performing the following command:

	sudo git clone git://git.osmocom.org/pysim.git

**NOTE: If you don’t already have ‘git’ on your host, you will need to perform a sudo apt-get install git before moving forward.**

The dependencies required by PySIM can be fetched by performing 

sudo apt-get install python-pyscard python-serial python-pip
sudo pip install pytlv

In order to program the SIM in its entirety, you’ll need a SIM card reader. Before moving forward, plug the card reader in your host’s USB port. Verify that your host has registered the use of the USB. If you have bought unlocked SIMs, your order should be accompanied by the cards’ specifications. If not, refer to the manufacturer to acquire this information. This is due to the fact that the PySIM commands are populated with the specifications of the card.

To our luck, we had a spreadsheet prepopulated with this information already at our disposal.  

**NOTE: Make sure you are in the pysim directory when you enter this command. Anything in brackets [ ] will need to be entered by you with information specific to your unique sim card.**

The template of the command is
./pySim-prog.py -p0 -t [card type] -a [ADM Pin] -x [mobile country code] -y [mobile network code] -i [IMSI] -s [iccid] -k [ki] -o [opc] -n [whatever name]

Applying the information specific to our cards, our command read:
./pySim-prog.py -p0 -t sysmoUSIM-SJS1 -a 49681225 -x 001 -y 01 -i 123400000056781 -s 1234511000000678901 -k E848CE041B42E17012B1F94B545CE623 -o DA8FBF229BAEA1D428472D42538067A3 -n myname

IMPORTANT: After three failed attempts of trying to program a SIM’s ADM key, the card will block you. If this happens to you, you will have to perform additional research in order to determine remediation steps. 

You can now place this SIM in any GSM compatible phone and the SIM be recognized.  If your BladeRF tower is transmitting, you can expect the newly programmed SIM/Cellular combo to connect within five to ten minutes.
Testing the Sim Cards with the BladeRF tower 
 


As demonstrated in the following images, we were able to connect multiple devices to the cellular network. We were prompted with a welcome message which assigned us our phone number. We texted our environment by exchanging text messages and exchanging phone calls.

We started simple first testing a text message. The number my device was assigned was 0010024384. Exchanging this phone number with my companion, they sent me a text message:




**NOTE: We found it easiest to perform this lab on old models of GSM. Unfortunately, many new phones come locked by the cell-phone provider. This inhibits one from equipping the phone with any SIM other than the one tailored to the cell provider. If your phone is locked by a cell-phone provider, best plan of action is to contact them requesting an unlock code. This would allow you to operate off your own sim card.**



Having verified the text message was a success, we attempted a phone call. We migrated to opposites of the room to ensure we didn’t talk over ourselves.

**Note:Since we have set the power of the BladeRF to be really low, the Tower’s range is very limited. In other words, don’t stray too far from the source.**

Dialing the number of another comrade on the network, we were able to establish a call.



As verified by the screenshots, the setup is complete. You should be able to replicate all that we’ve depicted right here. Enjoy connecting GSM phones and continue experimenting with the capacity of the cellular network! 

Two resources we found to be incredibly useful were those listed below. Fortunately, we weren’t the first ones to conquer this endeavor. 


### References

https://www.evilsocket.net/2016/03/31/how-to-build-your-own-rogue-gsm-bts-for-fun-and-profit/ 
    
https://github.com/goodboytower/GoodBTS
