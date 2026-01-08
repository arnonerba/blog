---
title: "How to Fix \"No Pattern File Found\" in Stata’s dm0082 Package"
modified_date: "2026-01-06"
last_modified_at: "2026-01-06"
---
If you're using the [dm0082](https://www.stata-journal.com/article.html?article=dm0082) package in Stata, you may encounter the following error when using commands such as `stnd_compname`:
```
No pattern file found in ~/ado/plus/p/
```

This error indicates that your installation of the dm0082 package is missing several important ancillary files. We specifically encountered this error on a Linux system, but you'll likely see a similar message on Windows or macOS.

The root cause of this error is that the ancillary files for dm0082 are not automatically placed in the correct location when they are installed. Instead, they are copied to your current working directory. The following pattern files will need to be moved from your current working directory to the folder referenced in the error message above:
```
P10_namecomp_patterns.csv
P21_spchar_namespecialcases.csv
P22_spchar_remove.csv
P23_spchar_rplcwithspace.csv
P30_std_entity.csv
P40_std_commonwrd_name.csv
P50_std_commonwrd_all.csv
P60_std_numbers.csv
P70_std_nesw.csv
P81_std_smallwords_all.csv
P82_std_smallwords_address.csv
P90_entity_patterns.csv
P110_std_streettypes.csv
P120_pobox_patterns.csv
P131_std_secondaryadd.csv
P132_secondaryadd_patterns.csv
```

On Linux, the proper destination for these files will likely be `~/ado/plus/p/`. If you are on Windows, it will likely be `c:\ado\plus\p\` instead.

**Note:** Make sure that the name of each pattern file starts with a capital "P" rather than a lowercase "p". The filenames may change to all lowercase when the ancillary files are downloaded. If that happens to you and you're running Linux, you can use the `rename` command as described in [this Ask Ubuntu post](https://askubuntu.com/questions/1069263/changing-first-letter-of-a-filename-to-uppercase) to quickly fix all the filenames at once.

---

**References:**
- [Fuzzy match in Stata - Kai Chen](https://www.kaichen.work/?p=291)

---

**Bonus Section:** If you want to confirm whether the folder referenced in the error message is correct, you can find your local Stata PLUS directory by running the `sysdir` command:
```stata
. sysdir
   STATA: /usr/local/stata17/
    BASE: /usr/local/stata17/ado/base/
    SITE: /usr/local/ado/
    PLUS: ~/ado/plus/
PERSONAL: ~/ado/personal/
OLDPLACE: ~/ado/
```

More info about the PLUS directory from the [Stata FAQ](https://www.stata.com/support/faqs/programming/personal-ado-directory/):
> PLUS is where Stata installs ado-files from the SJ and STB and ado-files that you have downloaded from the Internet through the help system or with the net command.
