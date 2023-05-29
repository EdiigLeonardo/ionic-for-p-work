# Introduction

The current document provides steps to create a personal [Ionic](https://ionicframework.com/) app that be installed using a free account in [Always Data](https://www.alwaysdata.com).

The user ***userx*** will correspond to the user that will be created by you.

The [Always Data](https://www.alwaysdata.com) account can also host web sites made with:
 * PHP
 * Python
 * Ruby
 * Node.js
 * .NET

When an Andoid or iOS app is ready to published to a store the steps mentioned [here for Android](https://ionicframework.com/docs/deployment/play-store) and [here for iOS](https://ionicframework.com/docs/deployment/app-store) must be followed. 

To allow the installation of debug APK's in an Android smartphone we need to follow the steps [here](https://docs.bravostudio.app/app-publication/publishing-your-app/android-publication-complete-process/3.-install-the-debug-apk-in-your-device).

## Create Always Data account 
 * Go to to [Always Data](https://www.alwaysdata.com) and click on "100 MB plan Free for life"
 * Create an account using a personal email 
 * After the account is created go to the [Always Data admin page](https://admin.alwaysdata.com/login/?next=/site/) 
   * In ***Remote access*** select ***SSH*** and in the settings area enable ***Enable password login***

### Create SSH keys to access the always data remote with using a password
```sh
cd ~
ssh-keygen
cat ~/.ssh/id_rsa.pub # see the content of the public key
ssh userx@ssh-vitor.alwaysdata.net
mkdir -p ~/.ssh
cd ~/.ssh
nano authorized_keys # copy the content of id_rsa.pub
chmod 0600 authorized_keys
exit 
ssh userx@ssh-vitor.alwaysdata.net # login without password
# if successful we can then use SFTP or scp to copy files without inputing the password
```

## Setup android environment in the Debian virtual machine
```sh
cd ~
# Delete old files and folders if they exit
rm -rf android/
rm -rf gradle-8.1/
rm -rf jdk-17.0.7+7/
rm gradle-8.1-bin.zip
rm OpenJDK17U-jdk_x64_linux_hotspot_17.0.7_7.tar.gz
rm commandlinetools-linux-6858069_latest.zip
# Install gradle
wget https://services.gradle.org/distributions/gradle-8.1-bin.zip
unzip gradle-8.1-bin.zip
# Update required software
sudo apt install -y curl sudo wget net-tools nodejs unzip
# Install OpenJDK 17
wget https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.7%2B7/OpenJDK17U-jdk_x64_linux_hotspot_17.0.7_7.tar.gz
tar xvzf OpenJDK17U-jdk_x64_linux_hotspot_17.0.7_7.tar.gz
# Install Android command line tools
wget https://dl.google.com/android/repository/commandlinetools-linux-6858069_latest.zip
mkdir ~/android
unzip commandlinetools-linux-6858069_latest.zip
mv cmdline-tools/ ~/android/
cd ~/android/cmdline-tools/ && mkdir latest && mv bin/ lib/ NOTICE.txt  source.properties latest/
# Set environment variables
export ANDROID_HOME=$HOME/android/
export ANDROID_SDK_ROOT=$HOME/android/
export PATH=/usr/local/bin:/usr/bin:/bin:$HOME/jdk-17.0.7+7/bin/:$HOME/gradle-8.1/bin/
export JAVA_HOME=$HOME/jdk-17.0.7+7/
# Add environment variables to .bashrc
echo "export ANDROID_HOME=$HOME/android/" >> ~/.bashrc
echo "export ANDROID_SDK_ROOT=$HOME/android/" >> ~/.bashrc
echo "export PATH=/usr/local/bin:/usr/bin:/bin:$HOME/jdk-17.0.7+7/bin/:$HOME/gradle-8.1/bin/" >> ~/.bashrc
echo "export JAVA_HOME=$HOME/jdk-17.0.7+7/" >> ~/.bashrc
. ~/.bashrc
# Install required android components
$HOME/android/cmdline-tools/latest/bin/sdkmanager --update
$HOME/android/cmdline-tools/latest/bin/sdkmanager --version
yes | $HOME/android/cmdline-tools/latest/bin/sdkmanager \
  --install "build-tools;30.0.3"
yes | $HOME/android/cmdline-tools/latest/bin/sdkmanager \
  --install "platforms;android-30"
yes | $HOME/android/cmdline-tools/latest/bin/sdkmanager \
  --install "system-images;android-30;google_apis_playstore;x86"
# check if licenses are accepted
$HOME/android/cmdline-tools/latest/bin/sdkmanager --licenses 
# Check java and gradle version
java -version
gradle -version

```


## Create Ionic test project in the Debian virtual machine
```sh
cd ~
rm -rf IonicTest/ 
rm -rf node_modules/
npm install @ionic/cli
echo -e "\n" | node_modules/.bin/ionic start IonicTest tabs --type=angular --capacitor
# choose NgModules
cd ~/IonicTest
npm i
npm install @ionic/cli cordova-res
# Test in chromium browser
node_modules/.bin/ionic serve 
# In a new terminal tab run chromium
chromium --disable-web-security --disable-gpu --user-data-dir="/tmp" http://localhost:8100
# Stop ionic serve with Ctrl+C
# Build APK for Android 
node_modules/.bin/ionic capacitor add android 
npm run build # create assets 
# share pseudonymous usage data: No 
npx cap sync
cd android/
gradle build # Build APK
# Copy APK to the always data remote host
scp ./app/build/outputs/apk/debug/app-debug.apk userx@ssh-userx.alwaysdata.net:/home/userx/www

# Go to the remote host and edit the index.html so it points to the copied APK
ssh userx@ssh-userx.alwaysdata.net
cd www 
nano index.html
```
 * In the smartphone goto https://userx.alwaysdata.net/app-debug.apk and install the app
 * Or go to https://userx.alwaysdata.net/ anc click on the anchor that points to the APK

## Update the APK
```sh
# Change text
cd ~/IonicTest
code . & 
# Open src/app/explore-container/explore-container.component.html
# Add text "stuff made"
cd ~/IonicTest
npx cap sync
cd android/
gradle build
# Copy APK to the always data remote host
scp ./app/build/outputs/apk/debug/app-debug.apk userx@ssh-userx.alwaysdata.net:/home/userx/www
```
 * In the smartphone goto https://userx.alwaysdata.net/app-debug.apk and install the app
 * Or go to https://userx.alwaysdata.net/ anc click on the anchor that points to the APK
