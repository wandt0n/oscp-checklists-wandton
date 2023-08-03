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
testscript.sh will be executed every month $(*)$ once every 3 days $(*/3)$ at 07:05 and 07:35 
