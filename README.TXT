###### Ubuntu Main Repos
deb http://us.archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse

###### Ubuntu Update Repos
deb http://us.archive.ubuntu.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://us.archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://us.archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse

###### Ubuntu Partner Repo
deb http://archive.canonical.com/ubuntu bionic partner
http://ftp.yzu.edu.tw/Linux/linuxlite/
#! /bin/bash
#--------------------------------------------------------------------------------------------------------
# Name: Lite Sources
# Description: Select Repository Mirror on Linux Lite.
# Authors: Jerry Bezencon, Ralphy
# Website: https://www.linuxliteos.com
#--------------------------------------------------------------------------------------------------------

# Application icon
_APPICON="/usr/share/icons/Papirus/24x24/apps/litesources.png"
_IMAGEICON="/usr/share/icons/Papirus/48x48/apps/litesources.png"

# Flags folder
_APPFLAG="/usr/share/liteappsicons/litesources/flags/"

_APPNAME=" Lite Sources"
_SLIST="/etc/apt/sources.list.d/linuxlite.list"
TRF="/tmp/litesourcestemp"

if [ $EUID -ne 0 ]; then pkexec $0; exit 0; if [ "${PIPESTATUS[@]}" -eq "126" ]; then exit 0 ; fi; else :; fi

# The loop
while (true); do
    test -e ["$TRF"] || rm -f "$TRF"

# Repositories
aaa="http://mirror.unlockforus.com/linuxlite/"
bra="http://sft.if.usp.br/linuxlite/"
dea="http://mirror.alpix.eu/linuxliteos/"
dec="https://mirror.alpix.eu/linuxliteos/"
eca="http://mirror.cedia.org.ec/linuxliteos/"
ecb="http://mirror.ueb.edu.ec/linuxliteos/"
ecc="http://mirror.uta.edu.ec/linuxliteos/"
ecd="http://mirror.espoch.edu.ec/linuxliteos/"
enga="http://www.mirrorservice.org/sites/repo.linuxliteos.com/linuxlite/"
fra="http://linuxlite.3load.com/"
gra="http://ftp.cc.uoc.gr/mirrors/linux/linuxlite/"
hka="http://mirror-hk.koddos.net/linuxlite/"
hkb="https://mirror-hk.koddos.net/linuxlite/"
hua="http://quantum-mirror.hu/mirrors/pub/linuxlite/"
hub="https://quantum-mirror.hu/mirrors/pub/linuxlite/"
lua="http://mirror.unlockforus.com/linuxlite/"
nca="http://mirror.lagoon.nc/pub/linuxlite/"
nla="http://mirror.koddos.net/linuxlite/"
nlb="https://mirror.koddos.net/linuxlite/"
sea="http://ftp.acc.umu.se/mirror/linuxliteos.com/"
twa="http://ftp.yzu.edu.tw/Linux/linuxlite/"
usa="http://mirror.unlockforus.com/linuxlite/"
usb="http://mirror.unlockforus.com/linuxlite/"
usc="http://repo.linuxliteos.com/linuxlite/"
vna="https://mirror.freedif.org/LinuxLiteOS/"

# Check the release version
LLVER=$(awk '{print $3}' /etc/llver)
RELNUM=$( echo "$LLVER / 1" | bc )  # Get Release number
if [ "$RELNUM" == "3"  ]; then
	rel_version=citrine
elif [ "$RELNUM" == "4"  ]; then
	rel_version=diamond
fi

# Start splash
yad --width=400 --height=20 --info --skip-taskbar --undecorated --no-buttons --timeout=1 \
 		--text-align=center --text='\n\n\n\n<span font="Sans 14">Loading Lite Repositories</span>\n<span font="Sans 11"> \nplease wait...\n</span>' & sleep .7

# Create main dialog from repositories.txt file
items=()
while IFS='|' read -r num flag country region url _; do
    items+=( "$num" "$_APPFLAG/$flag.png" "  $country " "  $region" "  $url" )
		echo $url >> $TRF
done < <(sort -t'|' -k3 /usr/local/sbin/repositories)

# Show main dialog
function mirrors {
selected=$(yad --list --dclick-action= --title="$_APPNAME" --window-icon=$_APPICON --resize --always-print-result \
			    --width=720 --height=525 --image=$_IMAGEICON --image-on-top --text-align=left \
          --text='\n<span font="Sans 12">Select a Repository Mirror from the list below</span>\n
First, click on <b>Check Repository Status</b>.
Then select from one of the Repositories below.\n
* Anycast mirrors automatically select a mirror closest to you.\n' \
          --button=" Check Repository Status"!gtk-execute:2  --button=" Quit"!gtk-quit:1 --button=" Use selected Repository"!gtk-ok:0 \
          --separator= --column="NUM" --column=" Flag :IMG" --column=" Country" --column=" Region" --column=" URL" \
          --hide-column=1 --print-column=1 \
          "${items[@]}" )
opt=$?

# Main window buttons
# Exit on button QUIT
if [[ "$opt" -eq "1" ]]; then
  rm -f "$TRF" & exit
fi

# Execute repo check (status.sh)
if [[ "$opt" -eq "2" ]]; then
  xterm -geometry 94x32 -fa monaco -fs 13 -title "Lite Repository Mirrors - Status" -e "/usr/local/sbin/litesources-repo-check"
  rm -f "$TRF" && bash $0
  exit
fi

# Exit on button X
if [[ "$opt" -eq "252" ]]; then
  rm -f "$TRF"
  exit
fi
}

# Call main dialog function on execution
mirrors

# Update cache after switching mirror
update_cache() {
  yad --width=400 --info --on-top --skip-taskbar --undecorated --no-buttons --timeout=1 --timeout-indicator=bottom --text-align=center \
  		--text='\n\n\n<span font="Sans 12"> Switching mirror. Please wait...</span>\n'

  apt-get update | stdbuf -oL sed -n -e '/\[*$/ s/^/# /p' -e '/\*$/ s/^/# /p' | yad --width=650 --progress --pulsate --on-top --skip-taskbar --undecorated --no-buttons \
    --auto-close --text-align=center --text='\n\n\n<span font="Sans 12"> Updating cache. Please wait...</span>\n'
  if [ "${PIPESTATUS[0]}" -ne "0" ]; then
    zenity --info --title="  $_APPNAME" --width="390" --height="60" --ok-label="Close" \
           --text="\n$_APPNAME was unable to update the cache.\nThis may just be a temporary connectivity issues with the update server." 2>/dev/null
  fi
}

# Don't switch repo message
#no_switch() {
#  zenity --info --title="  $_APPNAME" --width="390" --height="60" --ok-label="Close" \
#         --text="\nThe selected mirror is already the default. No changes needed." 2>/dev/null
#}

no_switch() {
  yad --info --skip-taskbar --title="" --fixed --button="Close"\!gtk-close:0 --borders=10 --width="420" --height="60" --ok-label="Close" --image-on-top \
      --image="/usr/share/liteappsicons/litesources/lite-repo48.png" --text="\nThe selected mirror is already the default.\n"
}

# Switch mirrors
if [[ "$selected" == "aaa" || "$selected" == "lua" || "$selected" == "usa" || "$selected" == "usb" ]]; then
  if grep -q -F "$aaa" "$_SLIST"; then no_switch; else
		echo "deb $aaa $rel_version main" > "$_SLIST"
		update_cache;  zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$aaa" 2>/dev/null
  fi

elif [ "$selected" == "bra" ]; then
  if grep -q -F "$bra" "$_SLIST"; then no_switch; else
		echo "deb $bra $rel_version main" > "$_SLIST"
		update_cache;  zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$bra" 2>/dev/null
  fi

elif [ "$selected" == "dea" ]; then
  if grep -q -F "$dea" "$_SLIST"; then no_switch; else
    echo "deb $dea $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$dea" 2>/dev/null
  fi

elif [ "$selected" == "dec" ]; then
  if grep -q -F "$dec" "$_SLIST"; then no_switch; else
    echo "deb $dec $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$dec" 2>/dev/null
  fi

elif [ "$selected" == "eca" ]; then
  if grep -q -F "$eca" "$_SLIST"; then no_switch; else
		echo "deb $eca $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$eca" 2>/dev/null
  fi

elif [ "$selected" == "ecb" ]; then
  if grep -q -F "$ecb" "$_SLIST"; then no_switch; else
    echo "deb $ecb $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$ecb" 2>/dev/null
  fi
elif [ "$selected" == "ecc" ]; then
  if grep -q -F "$ecc" "$_SLIST"; then no_switch; else
		echo "deb $ecc $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$ecc" 2>/dev/null
  fi

elif [ "$selected" == "ecd" ]; then
  if grep -q -F "$ecd" "$_SLIST"; then no_switch; else
    echo "deb $ecd $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$ecd" 2>/dev/null
  fi

elif [ "$selected" == "enga" ]; then
  if grep -q -F "$enga" "$_SLIST"; then no_switch; else
    echo "deb $enga $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$enga" 2>/dev/null
  fi

elif [ "$selected" == "fra" ]; then
  if grep -q -F "$fra" "$_SLIST"; then no_switch; else
    echo "deb $fra $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$fra" 2>/dev/null
  fi

elif [ "$selected" == "gra" ]; then
  if grep -q -F "$gra" "$_SLIST"; then no_switch; else
    echo "deb $gra $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$gra" 2>/dev/null
  fi

elif [ "$selected" == "hka" ]; then
  if grep -q -F "$hka" "$_SLIST"; then no_switch; else
    echo "deb $hka $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$hka" 2>/dev/null
  fi

elif [ "$selected" == "hkb" ]; then
  if grep -q -F "$hkb" "$_SLIST"; then no_switch; else
    echo "deb $hkb $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$hkb" 2>/dev/null
  fi

elif [ "$selected" == "hua" ]; then
  if grep -q -F "$hua" "$_SLIST"; then no_switch; else
    echo "deb $hua $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$hua" 2>/dev/null
  fi

elif [ "$selected" == "hub" ]; then
  if grep -q -F "$hub" "$_SLIST"; then no_switch; else
    echo "deb $hub $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$hub" 2>/dev/null
  fi

elif [ "$selected" == "nca" ]; then
  if grep -q -F "$nca" "$_SLIST"; then no_switch; else
    echo "deb $nca $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$nca" 2>/dev/null
  fi

elif [ "$selected" == "nla" ]; then
  if grep -q -F "$nla" "$_SLIST"; then no_switch; else
    echo "deb $nla $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$nla" 2>/dev/null
  fi

elif [ "$selected" == "nlb" ]; then
  if grep -q -F "$nlb" "$_SLIST"; then no_switch; else
    echo "deb $nlb $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$nlb" 2>/dev/null
  fi

elif [ "$selected" == "sea" ]; then
  if grep -q -F "$sea" "$_SLIST"; then no_switch; else
    echo "deb $sea $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$sea" 2>/dev/null
  fi

elif [ "$selected" == "twa" ]; then
  if grep -q -F "$twa" "$_SLIST"; then no_switch; else
    echo "deb $twa $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$twa" 2>/dev/null
  fi

elif [ "$selected" == "tha" ]; then
  if grep -q -F "$tha" "$_SLIST"; then no_switch; else
    echo "deb $tha $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$tha" 2>/dev/null
  fi

elif [ "$selected" == "usc" ]; then
  if grep -q -F "$usc" "$_SLIST"; then no_switch; else
    echo "deb $usc $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$usc" 2>/dev/null
  fi

elif [ "$selected" == "vna" ]; then
  if grep -q -F "$vna" "$_SLIST"; then no_switch; else
    echo "deb $vna $rel_version main" > "$_SLIST"
		update_cache; zenity --info --title="  $_APPNAME - New Mirror" --width="360" --height="60" --text="\nLinux Lite repository mirror has been changed to: \n\n$vna" 2>/dev/null
  fi
fi
done
exit 0
