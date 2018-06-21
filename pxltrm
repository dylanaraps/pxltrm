#!/usr/bin/env bash
#
# pxltrm - A pixel art editor for the terminal.

clear_screen() {
    printf '\e[2J\e[2;H\e[m'
    unset hist undos
}

status_line_clean() {
    printf '\e[s\e[%b;H\e[J\e[u' "$((LINES-3))"
}

get_term_size() {
    shopt -s checkwinsize; (:;:)
    [[ -z "${LINES:+$COLUMNS}" ]] && read -r LINES COLUMNS < <(stty size)
}

print_palette() {
    for i in {1..8}; do
        [[ "$i" == "$color" ]] && block_char="▃" || block_char=" "
        status+="\\e[48;5;${i}m\\e[30m${block_width// /${block_char}}\\e[m"
    done
}

status_line() {
    local status

    printf -v block_width "%$((COLUMNS / 24))s"
    printf -v padding '\e[%bC' "$((COLUMNS - COLUMNS / 3))"

    print_color
    print_palette

    hud="[d]raw, [e]rase, cle[a]r, [s]ave, [o]pen, [u]ndo, [r]edo"
    hud="${hud::$COLUMNS}"
    hud="${hud//\[/[\\e[1m}"

    printf '\e[s\e[;H\e[2K\e[m%b\e[%s;H\e[m%b\e[m, %b\e[m\e[99999D\e[A%b\e[u' \
           "${hud//\]/\\e[m\]}" \
           "$((LINES-2))" \
           "[\\e[1mc\\e[m]olor: \\e[1m${print_col}${color}" \
           "[\\e[1mb\\e[m]rush: \\e[1m${brush_char:=█}" \
           "${padding}${status//▃/ }\\n${padding}${status}"
}

save_file() {
    local IFS=
    printf '\e[2J\e[2;H%b\e[m\e[%s;H' "${hist[*]}" "$((LINES-3))"
}

load_file() {
    clear_screen
    printf '\e[2;H%b\e[2;H' "$(<"$1")"
    hist+=("$_")
}

undo() {
    undos+=("${hist[-1]}")
    printf '%b \b' "${hist[-1]}"
    unset 'hist[-1]'
}

redo() {
    hist+=("${undos[-1]}")
    printf '%b' "${undos[-1]}"
    unset 'undos[-1]'
}

hex_to_rgb() {
    ((r=16#${color:1:2},g=16#${color:3:2},b=16#${color:5:6}))
}

get_pos() {
    IFS='[;' read -p $'\e[6n' -d R -rs _ y x _
}

print_color() {
    case "${color:=7}" in
        "#"*) hex_to_rgb;: "\\e[38;2;${r};${g};${b}m" ;;
        [0-9]*):           "\\e[38;5;${color}m" ;;
        *) color="7";:     "\\e[38;5;7m" ;;
    esac
    printf -v print_col '%b' "$_"
}

paint() {
    get_pos
    printf '%b' "\\e[${y};${x}H${2}${1}\\b"
    hist+=("$_")
}

prompt() {
    printf '\e[s\e[%s;H\e[m' "$((LINES-1))"

    case "$1" in
        s) read -rp "save file: " f; save_file > "${f:-/dev/null}" ;;
        o) read -rp "load file: " f; [[ -f "$f" ]] && load_file "$f" ;;
        c) read -rp "input color: " color ;;
        b) read -rp "input brush: " brush_char ;;
        a) read -n 1 -rp "clear? [y/n]: " y; [[ "$y" == y ]] && clear_screen ;;
    esac

    printf '\e[u'
    status_line_clean
    status_line
}

cursor() {
    case "${1: -1}" in
        A|k) get_pos; ((y>2))         && printf '\e[A' ;;
        B|j) get_pos; ((y<LINES-4))   && printf '\e[B' ;;
        C|l) get_pos; ((x<COLUMNS-1)) && printf '\e[C' ;;
        D|h) printf '\e[D' ;;
        H)   printf '\e[999999D' ;;
        L)   printf '\e[999999C\e[D' ;;

        [1-8]) color="${1: -1}"; status_line ;;

        e) paint " " ;;
        d) print_color; paint "${brush_char:=█}" "$print_col" ;;
        u) (("${#hist}">0))  && undo; status_line ;;
        r) (("${#undos}">1)) && redo; status_line ;;

        a|b|c|o|s) prompt "${1: -1}" ;;
    esac
}

main() {
    clear_screen
    get_term_size
    status_line

    trap 'clear_screen' EXIT
    trap 'status_line_clean; get_term_size; status_line' SIGWINCH

    for ((;;)); { read -rs -n 1 key; cursor "$key"; }
}

main "$@"
