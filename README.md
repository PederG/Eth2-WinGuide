# Eth2-WinGuide
Guide to setting up Geth &amp; Prysm on Windows with autostart

This guide is currently a WIP. All attempts at staking are at your own risk.
I'm writing this mainly as an opportunity to learn and improve my own setup, but am happy to share my steps for anyone who can find them useful. 
Do not try to set up a node without understanding the steps and risk you are taking. I can not guarantee that these steps will work for you. 
Make sure you have read and understood all information @ https://launchpad.ethereum.org/

I strongly recommend that you learn on the tesnet before staking on mainnet. 

Thanks to /r/ethstaker for being a friendly & helpful community. 

This guide will curently not discuss topics that have been covered in detail elsewhere (hardware requirements etc.). I'm trying to focus on the steps that are specific for Windows and not commonly discussed. 

## Software
-Windows OS x64 (Tested on Windows 10 Pro & Windows Server 2019)  
-[NSSM](https://nssm.cc/)  
-[Geth](https://geth.ethereum.org/)  
-[Prysm](https://prysmaticlabs.com/)  

NSSM is the tool that I use to setup Geth & Prysm as a Windows Service to ensure they start at boot and relaunch if they crash. 

### Assumptions
The steps outlined here work for a specific (albeit common) scenario. If your setup differs you will have to adapt your steps.  
In this setup your PC only has 1 drive and that is the C: drive.  
I'm running Geth and Prysm on the same installation.  
This is a fresh installation and you haven't ran Geth or Prysm on it post installation.  

## Installation
### Geth
1. Grab the latest 64-bit installer from https://geth.ethereum.org/downloads/
2. Run the installer as admin, Next, Next, Finish

### NSSM
Download the latest release (2.24 at time of writing) from https://nssm.cc/download and unzip it to C:\

### Prysm
1. Open a Command Prompt as Administrator
2. Run the following commands one line at a time:
```
cd \
mkdir prysm && cd prysm
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.bat --output prysm.bat
reg add HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1
mkdir data
```

## Configuration
### Geth as a Service
1. Open a Command Prompt as Administrator
2. Run the following commands one line at a time:
```
mkdir \Geth
cd \nssm-2.24\win64
nssm.exe install "Geth Service" "C:\Program Files\Geth\geth.exe" 
nssm.exe set "Geth Service" AppParameters "--datadir c:\geth\data\ --http" 
nssm.exe set "Geth Service" DisplayName "Geth Service" 
nssm.exe set "Geth Service" AppStopMethodConsole 30000 
nssm.exe set "Geth Service" AppStdout "C:\Geth\GethLog.txt" 
nssm.exe set "Geth Service" AppStderr "C:\Geth\GethLog.txt" 
nssm.exe set "Geth Service" AppRotateFiles 1
```
3. To start Geth your can run ```C:\nssm-2.24\win64\nssm.exe start "Geth Service"``` or open Services and start it from there. From now on Geth will autostart at boot unless you disable the service. 

### Prysm Configuration Files
#### Beacon Node
1. Open a Command Prompt as Administrator
2. Run the following commands one line at a time:
```
cd \prysm\
echo http-web3provider: http://localhost:8545 >> beacon.yaml
echo accept-terms-of-use: true >> beacon.yaml
echo datadir: C:\prysm\data\beacon\ >> beacon.yaml
```
#### Validator
1. Open a Command Prompt as Administrator
2. Run the following commands one line at a time:
```
cd \prysm\
echo wallet-dir: C:\Wallets >> validator.yaml
echo wallet-password-file: C:\Wallets\WalletPW.txt >> validator.yaml
echo accept-terms-of-use: true >> validator.yaml
echo datadir: D:\prysm\data\validator >> validator.yaml
```
3. Import your validator key pairs. These are the keys you generated using the CLI tool from Launchpad.  
Remember to keep your Mnemonic safe. Without this your funds are lost forever.  
Replace C:\Path\to\validator_keys below so it points to your validator_keys folder  
```
mkdir \Wallets\
prysm.bat validator accounts import --keys-dir=C:\Path\to\validator_keys
```  
  Enter ```C:\Wallets``` as your wallet directory when asked, then pick a password for this wallet and lastly enter the password you picked when creating the key pair. 

4. Run these command but replace YOUR_WALLET_PASSWORD with the password you just picked. 
```
cd \Wallets\
echo YOUR_WALLET_PASSWORD >> WalletPW.txt
```
5. You can verify that the password was saved correctly by opening C:\Wallets\WalletPW.txt

### Beacon Node as a Service
1. Open a Command Prompt as Administrator
2. Run the following commands one line at a time:
```
cd \nssm-2.24\win64
nssm.exe install "Prysm Beacon Service" "C:\prysm\prysm.bat" 
nssm.exe set "Prysm Beacon Service" AppParameters "beacon-chain --config-file=C:\prysm\beacon.yaml" 
nssm.exe set "Prysm Beacon Service" DisplayName "Prysm Beacon Service" 
nssm.exe set "Prysm Beacon Service" AppStopMethodConsole 30000 
nssm.exe set "Prysm Beacon Service" AppStdout "C:\prysm\BeaconLog.txt" 
nssm.exe set "Prysm Beacon Service" AppStderr "C:\prysm\BeaconLog.txt" 
nssm.exe set "Prysm Beacon Service" AppRotateFiles 1
```
3. To start Beacon Node your can run ```C:\nssm-2.24\win64\nssm.exe start "Prysm Beacon Service"``` or open Services and start it from there. From now on Geth will autostart at boot unless you disable the service. 

### Validator as a Service
1. Open a Command Prompt as Administrator
2. Run the following commands one line at a time:
```
cd \nssm-2.24\win64
nssm.exe install "Prysm Validator Service" "C:\prysm\prysm.bat" 
nssm.exe set "Prysm Beacon Service" AppParameters "validator --config-file=C:\prysm\validator.yaml" 
nssm.exe set "Prysm Beacon Service" DisplayName "Prysm Validator Service" 
nssm.exe set "Prysm Beacon Service" AppStopMethodConsole 30000 
nssm.exe set "Prysm Beacon Service" AppStdout "C:\prysm\ValidatorLog.txt" 
nssm.exe set "Prysm Beacon Service" AppStderr "C:\prysm\ValidatorLog.txt" 
nssm.exe set "Prysm Beacon Service" AppRotateFiles 1
```
3. To start Validator your can run ```C:\nssm-2.24\win64\nssm.exe start "Prysm Validator Service"``` or open Services and start it from there. From now on Geth will autostart at boot unless you disable the service. 

## WIP
- Windows Firewall  
-
