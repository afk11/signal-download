#!/bin/bash

version="$1"
download_dir="$HOME/signal"
unverified_dir="$download_dir/unverified"
tmp_dir="$download_dir/tmp"
certify_key="29:F3:4E:5F:27:F2:11:B4:24:BC:5B:F9:D6:71:62:C0:EA:FB:A2:DA:35:AF:35:C1:64:16:FC:44:62:76:BA:26"

# Version must be provided
if [ "$version" = "" ]; then
    echo "Missing signal version!"
    exit 1
fi

# Download dir may be overridden
if [ "$SIGNAL_DOWNLOAD_DIR" != "" ]; then
    download_dir="$SIGNAL_DOWNLOAD_DIR"
fi

# Create working directories if missing
[ -d $download_dir ] || mkdir $download_dir
[ -d $unverified_dir ] || mkdir $unverified_dir
[ -d $tmp_dir ] || mkdir $tmp_dir

download_link="https://updates.signal.org/android/Signal-website-universal-release-$version.apk"

# Download file to unverified dir
wget -O "$unverified_dir/signal-$version.apk" $download_link 2>/dev/null
if [ $? != 0 ]; then
    echo "Failed to download APK - is URL scheme correct?"
    exit 1
fi

if [ ! -f "$unverified_dir/signal-$version.apk" ]; then
    echo "Internal error, failed to fiond file at $unverified_dir/signal-$version.apk"
    exit 1
fi

# Unzip cert
unzip -p $unverified_dir/signal-$version.apk META-INF/SIGNAL_S.RSA > $tmp_dir/$version.cert

# Decode certificate info
keyout=$(keytool -printcert -file $tmp_dir/$version.cert)
keyout_test=$?

# Cleanup cert, important stuff in memory
rm $tmp_dir/$version.cert

if [ $keyout_test != 0 ]; then
    echo "Invalid signature on JAR file"
    exit 1
fi

signing_sha256=$(echo "$keyout" | grep SHA256 | cut -d" " -f3)
if [ "$signing_sha256" != "$certify_key" ]; then
    echo "FAILED TO VERIFY APK"
    echo "      GOT $signing_sha256"
    echo " EXPECTED $certify_key"
    echo ""
    echo "Download link: $download_link"
    exit 1
fi

mv "$unverified_dir/signal-$version.apk" "$download_dir/signal-$version.apk"
echo "Verified APK. Wrote file to $download_dir/signal-$version.apk"
exit 0
