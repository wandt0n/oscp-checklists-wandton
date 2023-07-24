```
5,35 7 */3 * * testscript.sh
 ┬   ┬  ┬  ┬ ┬
 │   │  │  │ │
 │   │  │  │ └ day of the week (0-7, Sonntag ist 0 oder 7)
 │   │  │  └── month (1-12)
 │   │  └───── day of the month (1-31)
 │   └──────── hour (0-23)
 └──────────── minute (0-59)

```
testscript.sh wird in jedem Monat $(*)$ alle 3 Tage $(*/3)$
immer um 07:05 und 07:35 ausgeführt.
