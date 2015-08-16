# Maintainer: Det <nimetonmaili at gmail a-dot com>

pkgname=lastpass-universal
pkgver=2.0.20
_chromver=2.0.25-1     # The actual extensions' versions
_ffver=2.0.20-2
_opver=2.0.25-1      # This one's wrong to prevent redownloading it
pkgrel=4
pkgdesc="The Universal LastPass installer for Firefox, Chrome, and Opera"
arch=('any')
url="https://lastpass.com"
license=('custom')
makedepends=('unzip')
optdepends=('chromium: for Chromium'
            'chromium-dev: for Chromium (Dev Channel) (AUR)'
            'firefox: for Mozilla Firefox'
            'firefox-beta-bin: for Mozilla Firefox (Beta) (AUR)'
            'firefox-nightly: for Mozilla Firefox (Nightly) (AUR)'
            'google-chrome: for Google Chrome (AUR)'
            'google-chrome-beta: for Google Chrome (Beta Channel) (AUR)'
            'google-chrome-dev: for Google Chrome (Dev Channel) (AUR)'
            'opera: for Opera'
            'opera-next: for Opera Next (AUR)')
conflicts=('lastpass-plugin-opera')
install=$pkgname.install
#source=('https://download.lastpass.com/lplinux.tar.bz2') # Let's do this a bit more efficiently for now
source=("lpchrome_linux_$_chromver.crx"::"https://download.lastpass.com/lpchrome_linux.crx" # Chrome
        "lp_linux_$_ffver.xpi"::"https://download.lastpass.com/lp_linux.xpi"  # Firefox
        'profiles.ini'
        "lastpass-$_opver.oex"::'https://addons.opera.com/extensions/download/lastpass/' # Opera
        'prefs.dat'
        'widgets.dat')
noextract=("lp_linux_$_ffver.xpi" "lastpass-$_opver.oex")
md5sums=('132ed9bd99ed5720f17720bec8282368'  # lpchrome_linux_$_chromver.crx
         '9145a1e1bfb7fef6fae16d46f57a29dd'  # lp_linux_$_ffver.xpi
         '08f0ef46608ca1811029e43ef0650790'  # profiles.ini
         'ecfb0c9cb1788b99ed4252d57f0da9f6'  # lastpass-$_opver.oex
         '45733a855b9d2f0b507d6ed67b79d395'  # prefs.dat
         '92a87534f8c36a3e21ed128163b98945') # widgets.dat

package() {
  # Since we're installing to $HOME in .install, we need hacks such as this to
  # check whether some clever guy is building with --asroot
  if [ $USER = root ]; then
    msg2 "NOTE: Makepkg is running as root. Most files will be installed to /root."
    read -p "$(tput bold)$(tput setaf 4)  ->$(tput sgr0)$(tput bold) Continue? (Y/n) $(tput sgr0)"
    [[ $REPLY != [yY] ]] && error "Aborted by user! Exiting..." && echo && exit 1
  fi

  msg2 "Installing for Google Chromes/Chromiums"
  # This returns a warning about the extra bytes so redirect to /dev/null instead and use the || operator
  unzip -o lpchrome_linux_$_chromver.crx -d lpchrome &>/dev/null || true

  # Install to a single place and just link it for everybody else
  CRX=/usr/share/lastpass/lpchrome.crx
  JSON=hdokiejnpimakedhajhdlcegeplioahd.json
  echo "{ \"external_crx\": \"$CRX\", \"external_version\": \"${_chromver/-*}\" }" > $JSON
  for i in opt/google/chrome usr/share/chromium usr/share/chromium-dev; do
    install -Dm644 $JSON "$pkgdir"/$i/extensions/$JSON
  done
  install -Dm644 lpchrome_linux_$_chromver.crx "$pkgdir"/$CRX

  # Allow silent installation since Chrome 21: http://www.chromium.org/administrators/policy-list-3#ExtensionInstallSources
  echo "{ \"ExtensionInstallSources\": [\"https://lastpass.com/*\", \"https://*.lastpass.com/*\"] }" > lastpass_policy.json
  for i in opt/chrome chromium chromium-dev; do
    install -Dm644 lastpass_policy.json "$pkgdir"/etc/$i/policies/managed/lastpass_policy.json
  done

  msg2 "Installing for Mozilla Firefoxes"
  # The extension and the profiles.ini go to $HOME so do this in the .install
  for i in lp_linux_$_ffver.xpi profiles.ini; do
    install -Dm644 $i "$pkgdir"/usr/share/lastpass/$i
  done

  msg2 "Installing for Operas"
  # Same for Operas and their settings
  install -m644 lastpass-$_opver.oex {pref,widget}s.dat "$pkgdir"/usr/share/lastpass/

  # The binary plugin
  [ "$CARCH" = 'x86_64' ] && _64=64
  for i in opera opera-next; do
    install -Dm644 lpchrome/libnplastpass$_64.so "$pkgdir"/usr/lib/$i/plugins/libnplastpass$_64.so
  done

  # Write the extension versions and the $USER to the .install
  sed -e "s/_ffver= *[^ ]*/_ffver=$_ffver/" -e "s/_opver= *[^ ]*/_opver=$_opver/" \
      -e "s/_user= *[^ ]*/_user=$USER/" -i ../$pkgname.install
}