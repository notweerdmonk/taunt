#!/bin/bash

# taunt - it taunts you

# Messages here
declare -A messages=(
  ["NONZERO"]="non zero exit code"
  ["SIGSEGV"]="you have got a segmentation fault"
  ["SIGINT"]="you have got an interruption from terminal"
)

declare -A symbol_table=(
  ["NONZERO"]="-ne 0"
  ["MINUSONE"]=255
  ["-1"]=255
  ["255"]=255
)

declare -A signum_table=(
)

declare -A sysexit_table=(
)

declare -A exitcode_table=(
)

function build_tauntcode {
  declare -g g_tauntcode="\
function taunton {
    # Enable taunts
    [[ -n \"\${1}\" ]] && msgsfile=\"\$(realpath \"\${1}\")\" || unset msgsfile
    PATH=\"\${PATH}:"${g_selfpath}"\"
    cleanedprompt=\"\$(echo \"\${PROMPT_COMMAND}\" | sed -z 's|${g_selfpath}\s\+\\\$?;\s\+||g')\"
    export PREV_PROMPT_COMMAND=\"\${cleanedprompt}\"
    PROMPT_COMMAND=\"${g_selfpath} \\\$? \${msgsfile}; \${cleanedprompt}\"
}

function tauntoff {
    # Disable taunts
    cleanedpath=\"\$(echo \"\${PATH}\" | sed 's|:\?\s*${g_selfpath}\s*:\?||g')\"
    PATH=\"\${cleanedpath}\"
    cleanedprompt=\"\$(echo \"\${PROMPT_COMMAND}\" | sed 's|${g_selfpath}\s\+\\\$?\s\+.*;\s\+||g')\"
    PROMPT_COMMAND=\"\${cleanedprompt}\"
}

function _tauntcomp {
  # taunt autocompletion
  declare -g a opts=(
      -sym --sym -symbols --symbols
      -sig --sig -signals --signals -signum --signum
      -sys --sys -sysexit --sysexit
      -m --m -messages --messages
      -d --d -download --download
      -r --r -reg --reg -register --register
      -c --c -clean --clean
      -l --l -locale --locale
      -h --h -help --help
    )

  COMPREPLY=(\$(compgen -W \"\${opts[*]}\" -- \"\${COMP_WORDS[COMP_CWORD]}\"))
}
function _tauntcomp {
  declare -g a opts=(\
      -sym --sym -symbols --symbols \
	    -sig --sig -signals --signals -signum --signum \
	    -sys --sys -sysexit --sysexit \
	    -m --m -messages --messages \
	    -d --d -download --download \
	    -c --c -clean --clean \
	    -r --r -reg --reg -register --register \
	    --x --x -dereg --dereg -deregister --deregister \
	    -l --l -locale --locale \
	    -h --h -help --help \
    )

  cur=\"\${COMP_WORDS[COMP_CWORD]}\"
  prev=\"\${COMP_WORDS[COMP_CWORD - 1]}\"

  if [[ -d \"\${prev}\" ]] || \
    [[ \"\${prev}\" =~ ^-{1,2}(sym|symbols|messages|m|r|reg|register|dereg|deregister)\$ ]]
  then
    COMPREPLY=(\$(compgen -f -- \"\${cur}\"))
  elif [[ \"\${prev}\" =~ ^-{1,2}(l|locale)\$ ]]
  then
    COMPREPLY=(\$(compgen -W \"\$(locale -a)\" -- \"\${cur}\"))
  else
    COMPREPLY=(\$(compgen -W \"\${opts[*]}\" -- \"\${cur}\"))
  fi
}
complete -F _tauntcomp -o filenames ""$(basename "${g_selfpath}")"
}

function parse_signum {
  [[ -n "${1}" ]] && \
    {
      table_header "Signal" "Exit";
      len="${?}";
    }

  signum_generic_header="/usr/include/bits/signum-generic.h"
  [[ ! -f "${signum_generic_header}" ]] && \
    signum_generic_header="$(find /usr/include -type f -name "signum-generic.h")"

  signum_arch_header="/usr/include/bits/signum-arch.h"
  [[ ! -f "${signum_arch_header}" ]] && \
    signum_arch_header="$(find /usr/include -type f -name "signum-arch.h")"

  while IFS=" " read -r signame signum
  do
    signum_table["${signame}"]="${signum}"

    # Exit code is 128 + signal num in bash
    exitcode_table[$((${signum} + 128))]="${signame}"

    [[ -n "${1}" ]] && \
      printf "%-16s  %s\n" "${signame}" "$((${signum} + 128))"

    [[ -z "${messages[${signame}]}" ]] && g_linecount="$((${g_linecount} + 1))"

  done <<< \
    $(sed -rn 's/^#define\s+(SIG[A-Z0-9]+)\s+([0-9]{1,2}).*$/\1 \2/p' \
      "${signum_generic_header}" \
      "${signum_arch_header}")

  [[ -n "${1}" ]] && table_footer "${len}"
}

function parse_sysexit {
  [[ -n "${1}" ]] && \
    {
      table_header "Sysexit code" "Exit";
      len="${?}";
    }

  sysexits_header="/usr/include/sysexits.h"
  [[ ! -f "${sysexits_header}" ]] && \
    sysexits_header="$(find /usr/include -type f -name "sysexits.h")"

  while IFS=" " read -r codename codenum
  do
    sysexit_table["${codename}"]="${codenum}"
    exitcode_table["${codenum}"]="${codename}"

    [[ -n "${1}" ]] && \
      printf "%-16s  %s\n" "${codename}" "${codenum}"

    [[ -z "${messages[${codename}]}" ]] && g_linecount="$((${g_linecount} + 1))"

  done <<< \
    $(sed -rn 's/^#define\s+(EX_[A-Z0-9]+)\s+([0-9]{1,2}).*$/\1 \2/p' \
      "${sysexits_header}")

  [[ -n "${1}" ]] && table_footer "${len}"
}

function parse_symbols {
  [[ -n "${1}" ]] && \
    {
      table_header "Symbol" "Exit";
      len="${?}";
    }

  for name in "${!symbol_table[@]}"
  do
    exitcode_table["${symbol_table[${name}]}"]="${name}"

    [[ -n "${1}" ]] && \
      printf "%-16s  %s\n" "${name}" "${symbol_table[${name}]}"

    [[ -z "${messages[${name}]}" ]] && g_linecount="$((${g_linecount} + 1))"
  done

  [[ -n "${1}" ]] &&  table_footer "${len}"
}

function pull_messages {
  [[ -z "${2}" ]] && return -1

  msgs_file="${2}"

  if [[ -n "${1}" ]]
  then
    [[ "${1}" == "$(\. "sbegharf")" ]] && __="uggcf://tvguho.pbz/enqnerbet/enqner2/enj/ersf/urnqf/znfgre/qbp/sbegharf.sha" || url="${1}"
  else
    __="$(shuf -n1 -e "uggcf://cnfgrova.pbz/enj/uWHATPUf" "uggcf://tvguho.pbz/nyygbz/cebireo/enj/ersf/urnqf/znfgre/cebireof.gkg")"
  fi

  echo -n "Pulling messages from " && \
    ! ([[ -n "${url}" ]] && echo "${url}") && echo URL && read -r url < <(\. $__)

  [[ -z "${g_linecount}" || "${g_linecount}" -eq 0 ]] && \
    g_linecount="$(("${#signum_table[@]}" + "${#sysexit_table[@]}"))"

  [[ -z "${g_linecount}" || "${g_linecount}" -eq 0 ]] && g_linecount=50

  curl="$(command -v curl 2>/dev/null)"
  if [[ -n "${curl}" ]]
  then
    unset curl_exitcode
    msgs_str="$(curl --write-out "%{onerror}%{errormsg}" -sL ${url})"
    curl_exitcode="${?}"

    [[ "${curl_exitcode}" -ne 0 ]] && echo "${msgs_str}" && exit 0

    msgs_str="$(echo "${msgs_str}" | \
      sed -rn "s/^[0-9[:blank:][:punct:]]*([-_a-zA-Z0-9'\".,:;[:blank:]]+[^\x{0900}-\x{097F}]+)[\r\n]?$/\1/p" | \
      shuf -n${g_linecount} | \
      sed -rz 's/\r\n/\n/g'
    )"

    [[ -z "${msgs_str}" || "${msgs_str}" =~ ^[[:space:]]*$ ]] && \
      echo "Failed to download messages" && exit 0
 
    echo "${msgs_str}" > "${msgs_file}"
  fi
}

shopt -s extglob
function parse_messages {
  output=""
  for name in "${!messages[@]}"
  do
    exitcode="${signum_table[${name}]}"
    if [[ -z "${exitcode}" ]]
    then
      exitcode="${sysexit_table[${name}]}"
      [[ -z "${exitcode}" ]] && exitcode="${symbol_table[${name}]}"

    else
      exitcode="$((${exitcode} + 128))"
    fi

    [[ -n "${output}" ]] && output+=$'\n'
    output+="$(printf "%-16s  %-6s  %s\n" "${name}" "${exitcode}" "${messages[${name}]}")"
  done

  [[ -z "${1}" ]] && return -1

  while IFS=" " read -r signame message
  do
    message="${message#\'}"
    message="${message%\'}"
    message="${message#\"}"
    message="${message%\"}"
    message="${message#\'}"
    message="${message%\'}"

    case "${signame}" in
      SIG+([A-Z0-9]) )
        messages["${signame}"]="${message}"
        exitcode="$(("${signum_table["${signame}"]}" + 128))"
        name="${signame}"
        ;;

      EX_+([A-Z0-9]) )
        messages["${signame}"]="${message}"
        exitcode="${sysexit_table["${signame}"]}"
        name="${signame}"
        ;;

      +([0-9]) )
        messages["${signame}"]="${message}"
        exitcode="${signame}"
        exitcode_table["${exitcode}"]="${signame}"
        name="${signame}"
        ;;

      * )
        unset name

        for symbolname in "${!symbol_table[@]}"
        do
          [[ "${symbolname}" == "${signame}" ]] && name="${signame}" && \
            exitcode="${symbol_table["${name}"]}" && break
        done

        # We don't need signame anymore so prepend it to the message
        [[ -z "${name}" ]] && [[ -n "${signame}" ]] && message="${signame} ${message}"

        [[ -z "${name}" ]] && \
        for symbolname in "${!symbol_table[@]}"
        do
          [[ -z "${messages["${symbolname}"]}" ]] && name="${symbolname}" && \
            exitcode="${symbol_table["${name}"]}" && break
        done

        [[ -z "${name}" ]] && \
        for signalname in "${!signum_table[@]}"
        do
          [[ -z "${messages["${signalname}"]}" ]] && name="${signalname}" && \
            exitcode="${signum_table["${name}"]}" && break
        done

        [[ -z "${name}" ]] && \
        for sysexitname in "${!sysexit_table[@]}"
        do
          [[ -z "${messages["${sysexitname}"]}" ]] && name="${sysexitname}" && \
            exitcode="${sysexit_table["${name}"]}" && break
        done

        if [[ -z "${name}" ]]
        then
          name="MSG_${#messages[@]}"
          exitcode="${#exitcode_table[@]}"
        fi

        [[ -z "${message}" ]] && continue

        messages["${name}"]="${message}"

        exitcode_table["${exitcode}"]="${name}"

        symbol_table["${name}"]="${exitcode}"
        ;;

    esac

    line="$(printf "%-16s  %-6s  %s\n" "${name}" "${exitcode}" "${message}" | sed -r "s/'/\\\'/g" | sed -r 's/"//g')"

    if echo "${output}" | grep -oP "^${name}" > /dev/null 2>&1
    then
      output="$(echo "${output}" | sed -r "s/^${name}.*$/${line}/g")"
    else
      output+=$'\n'"${line}"
    fi

  done <<< "${!1}"

  echo "${output}"
}

function load_messages {
  msgs_file="/tmp/tauntfile"
  [[ -z "${HOME}" ]] || msgs_file="${HOME}/.cache/tauntfile" && \
    [[ -z "${USER}" ]] || msgs_file="/home/${USER}/.cache/tauntfile"

  code_file="/tmp/tauntcodes"
  [[ -z "${HOME}" ]] || code_file="${HOME}/.cache/tauntcodes" && \
    [[ -z "${USER}" ]] || code_file="/home/${USER}/.cache/tauntcodes"

  file="${g_argmap["readfile"]}"

  # Clean
  if [[ "$(("${g_argmask}" & 16))" -ne 0 ]]
  then
    ([[ ! -f "${msgs_file}" ]] || ! rm "${msgs_file}") && rm -f /tmp/tauntfile
    ([[ ! -f "${code_file}" ]] || ! rm "${code_file}") && rm -f /tmp/tauntcodes

    echo "Deleted downloads"
    shopt -u extglob
    exit 0

  # Download
  elif [[ "$(("${g_argmask}" & 32))" -ne 0 ]]
  then
    pull_messages "${g_argmap["url"]}" "${msgs_file}"
    # We can use msgs_str here
    echo "$(parse_messages msgs_str)" | tee "${code_file}"
    shopt -u extglob
    exit 0

  elif [[ -n "${file}" ]]
  then
    if [[ "${file}" == "-" ]]
    then
      while IFS= read -r line
      do
        msgs_str+="${line}"$'\n'
      done

    elif [[ -f "${file}" ]]
    then
      msgs_str="$(cat ${file})"
    fi
  fi

  [[ -f "${msgs_file}" ]] && msgs_str+="$(cat ${msgs_file})"

  [[ "$(("${g_argmask}" & 8))" -eq 0 ]] && redir="1>/dev/null" || \
    {
      table_header "Name" "Exit" "Message";
      len="${?}";
    }

  eval "parse_messages msgs_str "${redir}""

  [[ "$(("${g_argmask}" & 8))" -eq 0 ]] && redir="1>/dev/null" || \
    table_footer "${len}"

  [[ -z "${redir}" ]] && shopt -u extglob && exit 0
}

function find_bashrc {
  searchpath="/home/${USER}"
  [[ -n "${1}" ]] && searchpath="$(realpath "${1}")"

  echo -n "> Do you want to search (${searchpath}) ? [y/N/searchpath] : "
  read -r input

  [[ "${input}" =~ ^n|N$ ]] && exit 0

  if [[ ! "${input}" =~ ^y|Y$ ]]
  then
    input="$(realpath "${input}")"
    [[ -d "${input}" || -f "${input}" ]] && \
      searchpath="${input}"
  fi

  echo "Searching ${searchpath}"
  results=($(find "${searchpath}" -name ".bashrc" 2>/dev/null | sed -z 's/\n/ /g'))

  echo "Choose .bashrc:"
  for i in "${!results[@]}"
  do
    echo -e "  ${i}    ${results[${i}]}"
  done

  echo -n "Enter choice: "
  read choice

  if [[ "${choice}" =~ ^[0-9]*$ && \
    "${choice}" -ge 0 && "${choice}" -lt "${#results[@]}" ]]
  then
    bashrc="${results[${choice}]}"
    [[ -z "${bashrc}" ]] && exit 1

  else
    echo "Invalid choice"
    exit 1

  fi
}

function diff_vars {
  [[ -z "${1}" && -z "${2}" ]] && return -1

  unset choice
  [[ "$(("${g_argmask}" & 1024))" -eq 0 ]] && \
    {
      echo -n "> Do you want a diff ? [y/N] : ";
      read -r choice;
    }
  [[ "${choice}" =~ ^n|N$ ]] && return -1

  diffprog="diff"
  ! diffprog_actual="$(command -v "${diffprog}")" && \
    {
      err "diff not found";
      echo -n "> Suggest alternative program : ";
      read -r diffprog;
    } && \
    ! diffprog_actual="$(command -v "${diffprog}")" && \
      {
        err "${diffprog} not found";
        return -1;
      }

  [[ "${diffprog_actual}" =~ ^alias\ .+= ]] && \
    diffprog_actual="$(echo -n "${diffprog_actual}" | sed -nr "s/alias .*='(.*)'/\1/p")"

  [[ "${diffprog_actual}" =~ ^.*diff ]] && \
    diffprog_actual="${diffprog_actual} --color=auto -p --unified=5"

  ${diffprog_actual} <(echo "${1}" | sed '/^$/d') <(echo "${2}" | sed '/^$/d')
}

function register {
  [[ -n "${1}" ]] && bashrc="$(realpath "${1}")"

  [[ -z "${bashrc}" && ! -f "${bashrc}" ]] && \
    ([[ -z "${1}" ]] || wrn "${bashrc} not found") && \
    bashrc="/home/${USER}/.bashrc"

  if [[ ! -f "${bashrc}" ]]
  then
    err ".bashrc not found in home directory"

    find_bashrc
  fi

  info "Found ${bashrc}"

  unset choice
  [[ "$(("${g_argmask}" & 1024))" -eq 0 ]] && \
    {
      echo;
      echo -n "> Do you want to proceed ? [y/N] : ";
      read -r choice;
    }
  [[ "${choice}" =~ ^n|N$ ]] && exit 0

  info "Confirmed bashrc file for taunt registration"
  info "file: ${bashrc}"

  declare -g g_selfpath="$(realpath ${0})"
  build_tauntcode

  md5="$(echo -n "${g_tauntcode}" | sed '/^$/d' | md5sum | cut -d ' ' -f 1)"

  bashcode="$(sed -nr '/^function taunton \{$/,/^}$/p' "${bashrc}")"
  [[ -n "${bashcode}" ]] && \
    bashcode+=$'\n'"$(sed -nr '/^function _tauntcomp{$/,/complete -F _tauntcomp .*$/p' "${bashrc}")"

  [[ -n "${bashcode}" ]] && \
    oldmd5="$(echo -n "${bashcode}" | sed '/^$/d' | md5sum | cut -d ' ' -f 1)"

  [[ -n "${oldmd5}" ]] && \
    (
      [[ "${oldmd5}" == "${md5}" ]] && \
        {
          err "taunt already registered in ${bashrc}";
          exit 0;
        } || \
        {
          echo
          wrn "The checksums mismatch because the taunt functions in the bash script have "
          wrn "been modified. Unregister taunt to remove the functions."
          echo

          diff_vars "${g_tauntcode}" "${bashcode}"

          exit 0;
        }
    )

  [[ "$?" -eq 0 ]] && exit 0

  bashrc_orig="$(realpath ./bashrc_orig_${USER})"
  cp -f "${bashrc}" "${bashrc_orig}"
  info "Backed up bashrc"
  info "file: ${bashrc_orig}"

  echo "${g_tauntcode}" >> "${bashrc}"

  info "taunts registered successfully"
}

function deregister {
  [[ -n "${1}" ]] && bashrc="$(realpath "${1}")"

  [[ ! -f "${bashrc}" ]] && \
    ([[ -z "${1}" ]] || wrn "${bashrc} not found") && \
    bashrc="/home/${USER}/.bashrc"

  if [[ ! -f "${bashrc}" ]]
  then
    wrn ".bashrc not found in home directory"

    find_bashrc
  fi

  info "Found ${bashrc}"

  unset choice
  [[ "$(("${g_argmask}" & 1024))" -eq 0 ]] && \
    {
      echo;
      echo -n "> Do you want to proceed ? [y/N] : ";
      read -r choice;
    }
  [[ "${choice}" =~ ^n|N$ ]] && exit 0

  info "Confirmed bashrc file for taunt deregistration"
  info "file: ${bashrc}"

  declare -g g_selfpath="$(realpath ${0})"
  build_tauntcode

  md5="$(echo -n "${g_tauntcode}" | sed '/^$/d' | md5sum | cut -d ' ' -f 1)"

  bashcode="$(sed -nr '/^function taunton \{$/,/^}$/p' "${bashrc}")"
  [[ -n "${bashcode}" ]] && \
    bashcode+=$'\n'"$(sed -nr '/^function _tauntcomp{$/,/complete -F _tauntcomp .*$/p' "${bashrc}")"

  [[ -n "${bashcode}" ]] && \
    oldmd5="$(echo -n "${bashcode}" | sed '/^$/d' | md5sum | cut -d ' ' -f 1)" || \
    {
      err "taunt is not registered in ${bashrc}."; exit 0;
    }

  [[ -n "${oldmd5}" && "${oldmd5}" != "${md5}" ]] && \
    {
      echo
      wrn "The checksums mismatch because the taunt functions in the bash script have "
      wrn "been modified."
      echo

      diff_vars "${g_tauntcode}" "${bashcode}"

      unset choice
      [[ "$(("${g_argmask}" & 1024))" -eq 0 ]] && \
        {
          echo;
          echo -n "> Do you want to remove taunt functions ? [y/N] : ";
          read -r choice;
        }
      [[ "${choice}" =~ ^n|N$ ]] && exit 0
    }

  # TODO: change how this is used
  bashrc_modified="$(realpath ./bashrc_modified_${USER})"
  cp -f "${bashrc}" "${bashrc_modified}"
  info "Backed up modified bashrc"
  info "file: ${bashrc_modified}"

  sed -ri '/^# tauntcode [[:xdigit:]]+$/d' "${bashrc}"
  sed -ri '/^function taunton \{$/,/^}$/d; /^$/d' "${bashrc}"
  sed -ri '/^function tauntoff \{$/,/^}$/d; /^$/d' "${bashrc}"
  sed -ri '/^# tauntcode end$/d' "${bashrc}"

  info "taunts deregistered successfully"
}

locale_to_voice() {
  locale="${g_argmap["locale"]}"
  [[ -z "${locale}" ]] && locale="$(locale | grep -oP "(?<=^LANG=).*")"

  # Add your locale to voice mapping here
  [[ "en_US.UTF-8" == *"${locale}"* ]] && echo "gmw+en-US" && return # English (US)
  [[ "en_GB.UTF-8" == *"${locale}"* ]] && echo "gmw+en" && return # English (UK)
  [[ "es_ES.UTF-8" == *"${locale}"* ]] && echo "roa+es" && return # Spanish (Spain)
  [[ "es_MX.UTF-8" == *"${locale}"* ]] && echo "roa+es-mx" && return # Spanish (Latin America)
  [[ "fr_FR.UTF-8" == *"${locale}"* ]] && echo "roa+fr" && return # French (France)
  [[ "de_DE.UTF-8" == *"${locale}"* ]] && echo "gmw+de" && return # German (Germany)
  [[ "it_IT.UTF-8" == *"${locale}"* ]] && echo "roa+it" && return # Italian (Italy)
  [[ "nl_NL.UTF-8" == *"${locale}"* ]] && echo "gmw+nl" && return # Dutch (Netherlands)
  [[ "zh_CN.UTF-8" == *"${locale}"* ]] && echo "sit+cmn" && return # Chinese (Mandarin)
  [[ "ja_JP.UTF-8" == *"${locale}"* ]] && echo "jpx+ja" && return # Japanese
  [[ "ru_RU.UTF-8" == *"${locale}"* ]] && echo "zle+ru" && return # Russian
  [[ "hi_IN.UTF-8" == *"${locale}"* ]] && echo "inc+hi" && return # Hindi
  [[ "te_IN.UTF-8" == *"${locale}"* ]] && echo "dra+te" && return # Telugu
  [[ "ta_IN.UTF-8" == *"${locale}"* ]] && echo "dra+ta" && return # Tamil
  [[ "ml_IN.UTF-8" == *"${locale}"* ]] && echo "dra+ml" && return # Malayalam
  [[ "or_IN.UTF-8" == *"${locale}"* ]] && echo "inc+or" && return # Odia
  [[ "kn_IN.UTF-8" == *"${locale}"* ]] && echo "dr+kn" && return # Kannada
  [[ "pa_IN.UTF-8" == *"${locale}"* ]] && echo "inc+pa" && return # Punjabi
  [[ "ur_IN.UTF-8" == *"${locale}"* ]] && echo "inc+ur" && return # Urdu
  [[ "ko_KR.UTF-8" == *"${locale}"* ]] && echo "ko" && return # Korean
  [[ "vi_VN.UTF-8" == *"${locale}"* ]] && echo "aav+vi" && return # Vietnamese
  [[ "id_ID.UTF-8" == *"${locale}"* ]] && echo "poz+id" && return # Indonesian
  [[ "ms_MY.UTF-8" == *"${locale}"* ]] && echo "poz+ms" && return # Malay (Malaysia)
  [[ "fi_FI.UTF-8" == *"${locale}"* ]] && echo "urj+fi" && return # Finnish
  [[ "sv_SE.UTF-8" == *"${locale}"* ]] && echo "gmq+sv" && return # Swedish
  [[ "da_DK.UTF-8" == *"${locale}"* ]] && echo "gmq+da" && return # Danish
  [[ "is_IS.UTF-8" == *"${locale}"* ]] && echo "gmq+is" && return # Icelandic
  [[ "pl_PL.UTF-8" == *"${locale}"* ]] && echo "zlw+pl" && return # Polish
  [[ "cs_CZ.UTF-8" == *"${locale}"* ]] && echo "zlw+cs" && return # Czech
  [[ "sk_SK.UTF-8" == *"${locale}"* ]] && echo "zlw+sk" && return # Slovak
  [[ "hu_HU.UTF-8" == *"${locale}"* ]] && echo "urj+hu" && return # Hungarian
  [[ "ro_RO.UTF-8" == *"${locale}"* ]] && echo "roa+ro" && return # Romanian
  [[ "bg_BG.UTF-8" == *"${locale}"* ]] && echo "zls+bg" && return # Bulgarian
  [[ "sr_RS.UTF-8" == *"${locale}"* ]] && echo "zls+sr" && return # Serbian
  [[ "hr_HR.UTF-8" == *"${locale}"* ]] && echo "zls+hr" && return # Croatian
  [[ "sl_SI.UTF-8" == *"${locale}"* ]] && echo "zlw+sl" && return # Slovenian
  echo "gmw+en-US"
}

function find_espeak {
  espeak="$(command -v espeak-ng)"
  if [[ -z "${espeak}" ]]
  then
    err "espeak-ng not found"

    espeak="$(command -v espeak)"
    [[ -z "${espeak}" ]] && err "espeak not found" && exit 1
  fi

  [[ -n "${espeak}" ]] && g_espeakvoice="$(locale_to_voice)" && return

  exit 1
}

function exitcode_to_message {
  if [[ -n "${1}" ]]
  then
    name="${exitcode_table[${1}]}"
    if [[ -n "${name}" ]]
    then
      msg="${messages[${name}]}"
      [[ -n "${msg}" ]] && echo "${msg}" && return

    fi

    echo "${messages[NONZERO]}"
  fi
}

function speak {
  if [[ -z "${1}" ]]
  then
    msg="$(shuf -n1 -e "${messages[@]}")"

  else
    [[ "${1}" =~ ^0*$ ]] && return
    msg="$(exitcode_to_message ${1})"

  fi

  echo "taunting..."
  "${espeak}" -v "${g_espeakvoice}" -s 180 "${msg}"
  echo -ne "\033[A\033[K"
  echo "command exited with ${1}"
}

function parse_args {
  declare -gA g_argmap=()
  declare -g g_argmask=0
  declare -g g_argshift=0
  declare -g g_linecount=0
  declare -g g_espeakvoice="gmw+en-US"

  [[ "${1}" =~ ^-?[0-9]+$ ]] && \
    {
      g_argmap["exitcode"]="$(printf "%u" "$((${1} & 0xff))")";
      shift;
      g_argshift="$(("${g_argshift}" + 1))";
    } || .() { echo -n "${1}" | tr "A-Za-z" "N-ZA-Mn-za-m5-90-4-321"; }

  expect="readfile"
  while [[ "${g_argmask}" -lt "$((2**10 - 1))" && (-n "${expect}" || -n "${1}") ]]
  do
    case "${1}" in
      ?(-)?(-)@(sym|symbols) )
        if [[ "$(("${g_argmask}" & 1))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 1))
        fi
        expect="parsefile"
        ;;

      ?(-)?(-)@(sig|signals|signum) )
        if [[ "$(("${g_argmask}" & 2))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 2))
        fi
        expect=""
        ;;

      ?(-)?(-)@(sys|sysexit) )
        if [[ A"$(("${g_argmask}" & 4))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 4))
        fi
        expect=""
        ;;

      ?(-)?(-)@(messages|m) )
        if [[ "$(("${g_argmask}" & 8))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 8))
        fi
        expect="readfile"

        ;;
      ?(-)?(-)@(download|d) )
        if [[ "$(("${g_argmask}" & 32))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 32))
        fi
        expect="url"
        ;;

      ?(-)?(-)@(clean|c) )
        if [[ "$(("${g_argmask}" & 16))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 16))
        fi
        expect=""
        ;;

      ?(-)?(-)@(r|reg|register) )
        if [[ "$(("${g_argmask}" & 64))" -eq 0 && "$(("${g_argmask}" & 128))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 64))
        fi
        expect="bashrcfile"
        ;;

      ?(-)?(-)@(x|dereg|deregister) )
        if [[ "$(("${g_argmask}" & 128))" -eq 0 && "$(("${g_argmask}" & 64))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 128))
        fi
        expect="bashrcfile"
        ;;

      ?(-)?(-)@(l|locale) )
        if [[ "$(("${g_argmask}" & 256))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 256))
        fi
        expect="locale"
        ;;

      ?(-)?(-)@(h|help) )
        if [[ "$(("${g_argmask}" & 512))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 512))
        fi
        expect=""
        ;;

      ?(-)?(-)@(y|yes) )
        if [[ "$(("${g_argmask}" & 1024))" -eq 0 ]]
        then
          g_argmask=$((${g_argmask} | 1024))
        fi
        expect=""
        ;;

      * )
        [[ -n "${expect}" && -n "${1}" ]] && g_argmap["${expect}"]="${1}"
        expect=""
        ;;

    esac

    shift
    g_argshift="$(("${g_argshift}" + 1))"

  done
}

function print_help {
  selfname="$(realpath ${0})"
  selfname="$(basename ${selfname})"
  [[ -z "${selfname}" ]] && { echo "WARNING: Could not parse script name"; return; }

  echo -e "${selfname} - taunts on command exit codes"
  echo
  echo "USAGE"
  echo -e "\t${selfname} [OPTIONS...] [ARGUMENT]"
  echo
  echo "DESCRIPTION"
  echo -e "\tRead out messages to the user with espeak-ng based on the exit code of"
  echo -e "\tthe previous command in bash."
  echo
  echo "OPTIONS"
  echo -e "\t-sym, --sym, -symbols, --symbols [FILE]"
  echo -e "\t\tList user defined symbols with exit codes. Load messages from "
  echo -e "\t\tfile if provided and list all names including signals, standard "
  echo -e "\t\texit codes and unnamed messages with their names and exit codes "
  echo -e "\t\tafter parsing, in addition to user defined symbols for which "
  echo -e "\t\tmessages are not displayed alongside."
  echo
  echo -e "\t-sig, --sig, -signals, --signals, -signum, --signum"
  echo -e "\t\tList signals with exit codes that are read from system headers."
  echo
  echo -e "\t-sys, --sys, -sysexit, --sysexit"
  echo -e "\t\tList standard exit codes names with exit codes that are read "
  echo -e "\t\tfrom system header."
  echo
  echo -e "\t-m, --m, -messages, --messages [FILE]"
  echo -e "\t\tList messages with exit codes. Load messages from file if "
  echo -e "\t\tprovided."
  echo
  echo -e "\t-d, --d, -download, --download [URL]"
  echo -e "\t\tPull messages from URL over HTTP. The host should provide "
  echo -e "\t\tplaintext lines separated with linebreaks as response to a GET "
  echo -e "\t\trequest made using curl. Number of messages required equals "
  echo -e "\t\ttotal of unassigned signals and standard exit codes and as many "
  echo -e "\t\tlines are permuted at random from the HTTP response text. "
  echo -e "\t\tDownloaded text is stored on disk and the list of messages with "
  echo -e "\t\texit codes is also stored separately."
  echo
  echo -e "\t-r, --r, -reg, --reg, -register, --register [BASHRC]"
  echo -e "\t\tRegister bash commands to toggle taunts feature in the shell. "
  echo -e "\t\tUses the .bashrc file provied, otherwise chooses the .bashrc "
  echo -e "\t\tlocated in the user's home directory by default. On failure to "
  echo -e "\t\tfind the .bashrc there, the user's home directory is searched "
  echo -e "\t\tfor all files matching the name \".bashrc\", prompting the user"
  echo -e "\t\tto choose the desired file. The original .bashrc is backed up in"
  echo -e "\t\t the current directory. The script also searches for espeak-ng "
  echo -e "\t\ton the system falling back to espeak, else reporting failure."
  echo
  echo -e "\t-x, --x, -dereg, --dereg, -deregister, --deregister [BASHRC]"
  echo -e "\t\tDeregister bash commands to toggle taunts feature in the shell. "
  echo
  echo -e "\t-c, --c, -clean, --clean"
  echo -e "\t\tClear downloaded text and parsed messages."
  echo
  echo -e "\t-l, --l, -locale, --locale [LOCALE_HINT]"
  echo -e "\t\tSet the locale to be used for espeak voice selection. System "
  echo -e "\t\tlocale is used by default."
  echo
  echo -e "\t-h, --h, -help, --help"
  echo -e "\t\tDisplay usage information."
  echo
  echo -e "\t[FILE]"
  echo -e "\t\tLoad messages from file. See PARSING MESSAGES."
  echo
  echo "PARSING MESSAGES"
  echo -e "\tMessages which are formatted as \"<name> <message>\" where <name> matches"
  echo -e "\t a parsed signal or standard exit code or user defined symbols symbol, "
  echo -e "\tthey are assigned to respective names. Otherwise the messages get "
  echo -e "\tassigned to unassigned user defined symbols, followed by signals and "
  echo -e "\tfinally standard exit codes. Unused messages if any are assigned names of"
  echo -e "\tthe form \"MSG_XXXX\" where \"XXXX\" denotes the size of the table "
  echo -e "\tmapping exit codes to names and also the exit code for the message. After"
  echo -e "\tpulling messages from an URL and storing them on disk, the script only "
  echo -e "\tparses them. User may provide a file containing messages to be parsed."
  echo
  echo "AUTOCOMPLETE"
  echo -e "\ttaunt commands support argument autocompletion upon registraton. The "
  echo -e "\tscript should be sourced in the shell to enable autocompleton."
  echo
}


# Linux terminal pager provider
# Courtesy
# https://stackoverflow.com/a/46159778
# https://stackoverflow.com/users/14122/charles-duffy
pager() {
  # if stdout is not to a TTY, copy directly w/o paging
  [[ -t 1 ]] || { cat; return; }

  if [[ -n "${PAGER}" ]]; then  ## honor the user's choice, if they have a pager configured
    "${PAGER}"
  elif command -v bat >/dev/null 2>&1; then
    bat
  elif command -v less >/dev/null 2>&1; then
    less
  elif command -v more >/dev/null 2>&1; then
    more
  else
    echo "WARNING: No pager found; falling back to cat" >&2
    cat
  fi
}

# Logstuff
function check_color_term {
  colors="$(tput colors 2>/dev/null)" && \
    [[ "${colors}" -gt 2 ]] && \
    {
      declare -g ANSI_BLUE="\033[0;34m";    # Blue
      declare -g ANSI_GREEN="\033[0;32m";   # Green
      declare -g ANSI_YELLOW="\033[0;33m";  # Yellow
      declare -g ANSI_RED="\033[0;31m";     # Red
      declare -g ANSI_RESET="\033[0m";      # No Color
      return 0;
    }

  return -1
}

function info() {
  echo -e "${ANSI_GREEN}[*] $1${ANSI_RESET}"
}

function wrn() {
  echo -e "${ANSI_YELLOW}[-] $1${ANSI_RESET}"
}

function err() {
  echo -e "${ANSI_RED}[!] $1${ANSI_RESET}"
}

function table_header {
  printf "%-16s  %-6s  %s\n" "${1}" "${2}" "${3}"

  [[ -n "${3}" ]] && \
    len="$((24 + 2 + "${#3}"))" || \
    len="$((18 + "${#2}"))"

  for ((i=1; i<=len; i++))
  do
    echo -n -
  done
  echo

  return "${len}"
}

function table_footer {
  len="${1}"
  for ((i=1; i<=len; i++))
  do
    echo -n -
  done
  echo

  return "${len}"
}

function _tauntcomp {
  declare -g a opts=(\
      -sym --sym -symbols --symbols \
	    -sig --sig -signals --signals -signum --signum \
	    -sys --sys -sysexit --sysexit \
	    -m --m -messages --messages \
	    -d --d -download --download \
	    -r --r -reg --reg -register --register \
	    --x --x -dereg --dereg -deregister --deregister \
	    -c --c -clean --clean \
	    -l --l -locale --locale \
	    -h --h -help --help \
    )

  cur="${COMP_WORDS[COMP_CWORD]}"
  prev="${COMP_WORDS[COMP_CWORD - 1]}"

  if [[ -d "${prev}" ]] || \
    [[ "${prev}" =~ ^-{1,2}(sym|symbols|messages|m|r|reg|register|dereg|deregister)$ ]]
  then
    COMPREPLY=($(compgen -f -- "${cur}"))
  elif [[ "${prev}" =~ ^-{1,2}[l|locale]$ ]]
  then
    COMPREPLY=($(compgen -W "$(locale -a)" -- "${cur}"))
  else
    COMPREPLY=($(compgen -W "${opts[*]}" -- "${cur}"))
  fi
}

# Start
g_scriptname="$(basename "$(realpath "${BASH_SOURCE[0]}")")"
# Check for BASH_VERSION inside a subshell
([[ -n "{BASH_VERSION}" ]] && (return 0 2>/dev/null)) && \
  {
    autocomp_name="$(complete -p | grep "${g_scriptname}")"
    [[ -n "${autocomp_name}" ]] && autocomp_name="${autocomp_name##* }"
    [[ -n "${autocomp_name}" ]] && complete -r "${autocomp_name}"
    true
  } && \
  complete -F _tauntcomp -o filenames "${g_scriptname}" && \
  return 0

check_color_term

parse_args "$@"
shift "${g_argshift}"

[ "$(("${g_argmask}" & 512))" -ne 0 ] && pager <<< "$(print_help)" && exit 0

parse_symbols "$([ "$(("${g_argmask}" & 1))" -ne 0 ] && echo 1)"
[[ "$(("${g_argmask}" & 1))" -ne 0  && -f "${g_argmap["parsefile"]}" ]] && \
  msgs_str="$(cat "${g_argmap["parsefile"]}")" && \
  parse_messages msgs_str
parse_signum "$([ "$(("${g_argmask}" & 2))" -ne 0 ] && echo 1)"
parse_sysexit "$([ "$(("${g_argmask}" & 4))" -ne 0 ] && echo 1)"
shopt -u extglob

[[ "$(("${g_argmask}" & 7))" -ne 0 ]] && exit 0

# Register
if [[ "$(("${g_argmask}" & 64))" -ne 0 ]]
then
  find_espeak
  info "Found espeak at ${espeak}"
  register "${g_argmap["bashrcfile"]}"

elif [[ "$(("${g_argmask}" & 128))" -ne 0 ]]
then
  deregister "${g_argmap["bashrcfile"]}"

else
  find_espeak
  shopt -s extglob
  load_messages
  shopt -u extglob
  speak "${g_argmap["exitcode"]}"
fi
