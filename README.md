# mnps
هي اداه بايثون وظيفتها توليد اليوزرات
# -*- coding: utf-8 -*-
"""
MNPS - Professional Username Generator
Developed by: Morte Negra
Compatible with Pydroid 3 (no external libraries needed)
"""

import os
import re
import secrets
import string
import time

TOOL_NAME = "MNPS"
VERSION = "1.0"
AUTHOR = "Morte Negra"

LETTERS = string.ascii_lowercase
DIGITS = string.digits
VOWELS = "aeiou"
CONSONANTS = "bcdfghjklmnpqrstvwxyz"

PLATFORMS = {
    "1": ("Telegram", 5, 32, "_", False),
    "2": ("Instagram", 1, 30, "._", True),
    "3": ("TikTok", 2, 24, "._", True),
    "4": ("X / Twitter", 4, 15, "_", False),
    "5": ("General", 3, 20, "._", True),
}

MODES = {
    "1": "Random letters",
    "2": "Letters + digits",
    "3": "Pronounceable (looks like a real word)",
    "4": "Semi-pattern (e.g. xx_xx / x.x.x)",
    "5": "Based on your keyword",
    "6": "Repeated / stylish (aabb, abab)",
}


def banner():
    line = "=" * 40
    print(line)
    print(f"   {TOOL_NAME}  |  Username Generator  v{VERSION}")
    print(f"   Developed by {AUTHOR}")
    print(line)


def ask_int(prompt, lo, hi, default=None):
    while True:
        raw = input(prompt).strip()
        if raw == "" and default is not None:
            return default
        if raw.isdigit() and lo <= int(raw) <= hi:
            return int(raw)
        print(f"  [!] Enter a number between {lo} and {hi}.")


def ask_choice(prompt, options):
    while True:
        c = input(prompt).strip()
        if c in options:
            return c
        print("  [!] Invalid choice.")


def pick(chars):
    return secrets.choice(chars)


def gen_random_letters(n):
    return "".join(pick(LETTERS) for _ in range(n))


def gen_letters_digits(n):
    out = [pick(LETTERS)]
    out += [pick(LETTERS + DIGITS) for _ in range(n - 1)]
    return "".join(out)


def gen_pronounceable(n):
    out = []
    use_consonant = secrets.randbelow(2) == 0
    for _ in range(n):
        out.append(pick(CONSONANTS if use_consonant else VOWELS))
        use_consonant = not use_consonant
    return "".join(out)


def gen_semi_pattern(n, sep):
    if n < 3:
        return gen_random_letters(n)
    pos = secrets.randbelow(n - 2) + 1
    chars = [pick(LETTERS) for _ in range(n - 1)]
    chars.insert(pos, sep)
    return "".join(chars)


def gen_keyword(word, n):
    word = re.sub(r"[^a-z0-9]", "", word.lower())
    if not word:
        return gen_random_letters(n)
    styles = [
        lambda: word + pick(DIGITS) * 2,
        lambda: word + str(secrets.randbelow(999)),
        lambda: pick(LETTERS) + word,
        lambda: word + pick(LETTERS) + pick(LETTERS),
        lambda: "the" + word,
        lambda: word + "x",
        lambda: "i" + word,
        lambda: word + "_" + gen_random_letters(2),
        lambda: word + "." + gen_random_letters(2),
    ]
    return secrets.choice(styles)()


def gen_stylish(n):
    a, b = pick(LETTERS), pick(LETTERS)
    style = secrets.randbelow(4)
    if style == 0:
        base = a * 2 + b * 2          # aabb
    elif style == 1:
        base = (a + b) * 2            # abab
    elif style == 2:
        base = a + b + b + a          # abba
    else:
        base = a * 3 + b              # aaab
    while len(base) < n:
        base += pick(LETTERS)
    return base[:n]


def is_valid(name, min_len, max_len, allowed_sep, allow_dot_start_end=True):
    if not (min_len <= len(name) <= max_len):
        return False
    if not name[0].isalpha():
        return False
    if name[-1] in "._":
        return False
    if ".." in name or "__" in name or "._" in name or "_." in name:
        return False
    allowed = set(LETTERS + DIGITS + allowed_sep)
    return all(ch in allowed for ch in name)


def generate(mode, length, platform, keyword, count):
    _, min_len, max_len, seps, _ = platform
    results = []
    seen = set()
    attempts = 0
    limit = count * 200

    while len(results) < count and attempts < limit:
        attempts += 1
        if mode == "1":
            name = gen_random_letters(length)
        elif mode == "2":
            name = gen_letters_digits(length)
        elif mode == "3":
            name = gen_pronounceable(length)
        elif mode == "4":
            name = gen_semi_pattern(length, secrets.choice(seps))
        elif mode == "5":
            name = gen_keyword(keyword, length)
        else:
            name = gen_stylish(length)

        if name in seen:
            continue
        if not is_valid(name, min_len, max_len, seps):
            continue
        seen.add(name)
        results.append(name)

    return results


def save_file(names, platform_name):
    stamp = time.strftime("%Y%m%d_%H%M%S")
    filename = f"MNPS_{platform_name.split()[0]}_{stamp}.txt"
    folders = ["/storage/emulated/0/", os.getcwd()]
    for folder in folders:
        try:
            path = os.path.join(folder, filename)
            with open(path, "w", encoding="utf-8") as f:
                f.write(f"# {TOOL_NAME} v{VERSION} - Developed by {AUTHOR}\n")
                f.write("\n".join(names) + "\n")
            return path
        except Exception:
            continue
    return None


def main():
    history = []
    while True:
        banner()

        print("\nPlatform rules:")
        for k, v in PLATFORMS.items():
            print(f"  {k}) {v[0]}  (length {v[1]}-{v[2]})")
        p_key = ask_choice("\nChoose platform: ", PLATFORMS)
        platform = PLATFORMS[p_key]

        print("\nGeneration mode:")
        for k, v in MODES.items():
            print(f"  {k}) {v}")
        mode = ask_choice("\nChoose mode: ", MODES)

        keyword = ""
        if mode == "5":
            keyword = input("Enter your keyword: ").strip()

        length = ask_int(
            f"Username length ({platform[1]}-{platform[2]}) [default 5]: ",
            platform[1], platform[2], default=max(5, platform[1]),
        )
        count = ask_int("How many usernames (1-500) [default 20]: ", 1, 500, default=20)

        print("\n[*] Generating...\n")
        names = generate(mode, length, platform, keyword, count)

        if not names:
            print("[!] Could not generate valid usernames with these settings.")
        else:
            for i, n in enumerate(names, 1):
                print(f"{i:>3}. {n}")
            print(f"\n[+] Generated {len(names)} username(s).")
            history.extend(names)

            if input("\nSave to file? (y/n): ").strip().lower() == "y":
                path = save_file(names, platform[0])
                if path:
                    print(f"[+] Saved to: {path}")
                else:
                    print("[!] Could not save the file.")

        print(f"\n[i] Total generated this session: {len(history)}")
        if input("\nGenerate again? (y/n): ").strip().lower() != "y":
            break
        print("\n" * 2)

    print(f"\nThanks for using {TOOL_NAME} - by {AUTHOR}\n")


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print(f"\n\nExited. {TOOL_NAME} by {AUTHOR}")