#!/usr/bin/env bash
# Script: nordvpn-server-find
# Repository URL: https://github.com/mrzool/nordvpn-server-find
# First release: December 2017
# Last updated: April 2022
# Author: Mattia Tezzele <info@mrzool.cc>

# Dependencies check
if ((BASH_VERSINFO[0] < 4)); then
  echo 2>&1 "Error: This script requires at least Bash 4.0"
  exit 1
fi

type jq >/dev/null 2>&1 || {
  echo 2>&1 "Missing dependency: This script requires jq 1.5"
  exit 1
}

# Reset in case getopts has been used previously in the shell.
OPTIND=1

quiet=0
location=false
capacity=30 # Default capacity value is 30%
limit=20    # Default limit value is 20

iso_codes=(
  'ad' 'ae' 'af' 'ag' 'ai' 'al' 'am' 'ao' 'aq' 'ar' 'as' 'at' 'au' 'aw' 'ax'
  'az' 'ba' 'bb' 'bd' 'be' 'bf' 'bg' 'bh' 'bi' 'bj' 'bl' 'bm' 'bn' 'bo' 'bq'
  'br' 'bs' 'bt' 'bv' 'bw' 'by' 'bz' 'ca' 'cc' 'cd' 'cf' 'cg' 'ch' 'ci' 'ck'
  'cl' 'cm' 'cn' 'co' 'cr' 'cu' 'cv' 'cw' 'cx' 'cy' 'cz' 'de' 'dj' 'dk' 'dm'
  'do' 'dz' 'ec' 'ee' 'eg' 'eh' 'er' 'es' 'et' 'fi' 'fj' 'fk' 'fm' 'fo' 'fr'
  'ga' 'gb' 'gd' 'ge' 'gf' 'gg' 'gh' 'gi' 'gl' 'gm' 'gn' 'gp' 'gq' 'gr' 'gs'
  'gt' 'gu' 'gw' 'gy' 'hk' 'hm' 'hn' 'hr' 'ht' 'hu' 'id' 'ie' 'il' 'im' 'in'
  'io' 'iq' 'ir' 'is' 'it' 'je' 'jm' 'jo' 'jp' 'ke' 'kg' 'kh' 'ki' 'km' 'kn'
  'kp' 'kr' 'kw' 'ky' 'kz' 'la' 'lb' 'lc' 'li' 'lk' 'lr' 'ls' 'lt' 'lu' 'lv'
  'ly' 'ma' 'mc' 'md' 'me' 'mf' 'mg' 'mh' 'mk' 'ml' 'mm' 'mn' 'mo' 'mp' 'mq'
  'mr' 'ms' 'mt' 'mu' 'mv' 'mw' 'mx' 'my' 'mz' 'na' 'nc' 'ne' 'nf' 'ng' 'ni'
  'nl' 'no' 'np' 'nr' 'nu' 'nz' 'om' 'pa' 'pe' 'pf' 'pg' 'ph' 'pk' 'pl' 'pm'
  'pn' 'pr' 'ps' 'pt' 'pw' 'py' 'qa' 're' 'ro' 'rs' 'ru' 'rw' 'sa' 'sb' 'sc'
  'sd' 'se' 'sg' 'sh' 'si' 'sj' 'sk' 'sl' 'sm' 'sn' 'so' 'sr' 'ss' 'st' 'sv'
  'sx' 'sy' 'sz' 'tc' 'td' 'tf' 'tg' 'th' 'tj' 'tk' 'tl' 'tm' 'tn' 'to' 'tr'
  'tt' 'tv' 'tw' 'tz' 'ua' 'ug' 'uk' 'um' 'us' 'uy' 'uz' 'va' 'vc' 've' 'vg'
  'vi' 'vn' 'vu' 'wf' 'ws' 'ye' 'yt' 'za' 'zm' 'zw'
)

array_contains() {
  local array="$1[@]"
  local seeking=$2
  local in=1
  for element in "${!array}"; do
    if [[ $element == "$seeking" ]]; then
      in=0
      break
    fi
  done
  return $in
}

while getopts ":l:c:n:rhq" opt; do
  case "$opt" in
  h)
    echo
    echo "nordvpn-server-find -r|(-l LOCATION [-c CAPACITY=30] [-n LIMIT=20] [-q])"
    echo
    echo "-r   $(tput smul)recommended$(tput rmul)   just output the recommended server for your location and exit, ignoring other options"
    echo "-q   $(tput smul)quiet$(tput rmul)         just output the best result for the given location, ignoring the -n and the -c option"
    echo "-l   $(tput smul)location$(tput rmul)      2-letter ISO 3166-1 country code (ex: us, uk, de)"
    echo "-c   $(tput smul)capacity$(tput rmul)      current server load, integer between 1-100 (defaults to 30)"
    echo "-n   $(tput smul)limit$(tput rmul)         limits number of results, integer between 1-100 (defaults to 20)"
    echo
    exit 0
    ;;
  q)
    quiet=1
    limit=1
    ;;
  r)
    rec_server=$(curl --silent "https://nordvpn.com/wp-admin/admin-ajax.php?action=servers_recommendations" | jq -r '.[0].hostname')
    echo "$rec_server"
    exit 0
    ;;
  l)
    array_contains iso_codes "${OPTARG,,}" && location=${OPTARG,,} >&2 ||
      {
        echo >&2 "Invalid location parameter."
        echo >&2 "Please provide a valid ISO 3166-1 country code to -l."
        echo >&2 "(ex: -l us)"
        exit 1
      }
    ;;
  c)
    if [[ "$OPTARG" =~ ^[0-9]+$ ]] && [ "$OPTARG" -ge 1 -a "$OPTARG" -le 100 ]; then
      capacity=$OPTARG >&2
    else
      {
        echo >&2 "Invalid capacity parameter."
        echo >&2 "Please provide an integer between 1-100 to -c."
        echo >&2 "(ex: -c 90)"
        exit 1
      }
    fi
    ;;
  n)
    if [[ $quiet -eq 1 ]]; then
      limit=1
    elif [[ "$OPTARG" =~ ^[0-9]+$ ]] && [ "$OPTARG" -ge 1 -a "$OPTARG" -le 100 ]; then
      limit=$OPTARG >&2
    else
      {
        echo >&2 "Invalid limit parameter."
        echo >&2 "Please provide an integer between 1-100 to -n."
        echo >&2 "(ex: -n 50)"
        exit 1
      }
    fi
    ;;
  :)
    echo >&2 "Option -$OPTARG requires an argument."
    echo >&2 "Use -h to show the help."
    exit 1
    ;;
  \?)
    echo >&2 "Invalid option: -$OPTARG"
    echo >&2 "Use -h to show the help."
    exit 1
    ;;
  esac
done

if [[ "$location" == false ]]; then
  echo >&2 "A location parameter is required."
  echo >&2 "Please provide a valid location parameter using the -l option."
  echo >&2 "(ex: -l us)"
  echo >&2 "Use -h to show the help."
  exit 1
fi

if [[ $quiet -ne 1 ]]; then
  # Output the following only if writing to a terminal
  if [ -t 1 ]; then
    echo
    echo "Looking for servers located in $(tput bold)${location^^}$(tput sgr0) with server load lower than $(tput bold)$capacity%$(tput sgr0)..."
    echo
  fi
fi

servers=$(curl --silent https://nordvpn.com/api/server/stats)

# Declare and populate array
declare -A results
while IFS="=" read -r key value; do
  results[$key]="$value"
done < <(jq --compact-output -r --arg location "$location" --arg capacity "$capacity" --arg limit "$limit" \
  '[. |
  to_entries[] |
  {key: .key, value: .value.percent} |
  select(.value <= ($capacity|tonumber)) |
  select(.key|contains($location))] |
  sort_by(.value) |
  from_entries |
  to_entries |
  map("\(.key)=\(.value|tostring)") |
  limit(($limit|tonumber);.[])' <<<"$servers")

# Print out results

if [ ${#results[@]} -eq 0 ]; then
  if [[ $quiet -eq 1 ]]; then
    exit -1
  fi
  # Colored output only if writing to a terminal
  if [ -t 1 ]; then
    echo >&2 "$(tput setaf 1)No servers found :("
  else
    echo >&2 "No servers found :("
  fi
else
  for key in "${!results[@]}"; do
    if [[ $quiet -eq 1 ]]; then
      echo "$key"
    else
      echo -e "$(tput setaf 6 && tput bold)$key $(tput sgr0) ${results[$key]}%"
    fi
  done |
    # if [ -t 1] checks if script is writing to a terminal.
    # If not, strip ANSI color codes from text stream.
    awk '{print $NF,$0}' | sort -n | cut -f2- -d' ' | column -t |
    if [ -t 1 ]; then cat; else sed -r "s/[[:cntrl:]]\[[0-9]{1,3}m//g"; fi
fi
