    %let pgm=utl-cumulative-sums-by-group-using-base-sas-and-sql-sas-r-python-excel;

    %stop_submission;

    Cumulative sums by group using base sas and sql sas r python excel

    github
    https://tinyurl.com/ycxn2d6a
    https://github.com/rogerjdeangelis/utl-cumulative-sums-by-group-using-base-sas-and-sql-sas-r-python-excel

    communities.sas
    https://tinyurl.com/2p9ahtru
    https://communities.sas.com/t5/SAS-Programming/Calculating-cumulative-sum/m-p/831436#M328548

      CONTENTS

         1 sas datastep
         2 r sql
         3 python sql
         4 sas sql solution fails
           it does work if id x visit is unique you get a cartesian join)
         5 sas hash
           Keintz, Mark
           mkeintz@outlook.com


    THESE SOULTIONS ONLY WORK WITH UNIQUE ID, VISIT

    https://github.com/rogerjdeangelis/utl-an-interesting-sql-accumulator-by-group
    https://github.com/rogerjdeangelis/utl-cumulative-sums-in-proc-sql


    /**************************************************************************************************************************/
    /*        INPUT            |       PROCESS                                           |        OUTPUT                      */
    /*        ======           |       =======                                           |        =======                     */
    /*                         |                                                         |                                    */
    /* ID    VISIT    VALUE    |1 BASE SAS                                               |                                    */
    /*                         |==========                                               |                                    */
    /* n        6        6     |                                                         |                                    */
    /* n        7        7     | Sorted for documentation                                |   ID VISIT VALUE CUMM              */
    /* n        8        8     | putposes only                                           |                                    */
    /* n        9        9     |                 OUTPUT                                  |   n     1     1     1              */
    /* n       10       10     | ID  VISIT VALUE    CUM                                  |   n     1     1     2              */
    /* n        1        1     |                                                         |   n     2     2     4              */
    /* n        1        1     | n      1     1      1                                   |   n     3     3     7              */
    /* n        2        2     | n      1     1 1+1  2                                   |   n     4     4    11              */
    /* n        3        3     | n      2     2 2+2  4                                   |   n     5     5    16              */
    /* n        4        4     | n      3     3 4+3  7                                   |   n     6     6    22              */
    /* n        5        5     | n      4     4     11                                   |   n     7     7    29              */
    /* x        6       16     | n      5     5     16                                   |   n     8     8    37              */
    /* x        7       17     | n      6     6     22                                   |   n     9     9    46              */
    /* x        8       18     | n      7     7     29                                   |   n    10    10    56              */
    /* x        9       19     | n      8     8     37                                   |                                    */
    /* x       10       20     | n      9     9     46                                   |   x     1    11    11              */
    /* x        1       11     | n     10    10     56                                   |   x     1    11    22              */
    /* x        1       11     |                                                         |   x     2    12    34              */
    /* x        2       12     |                                                         |   x     3    13    47              */
    /* x        3       13     |                                                         |   x     4    14    61              */
    /* x        4       14     | proc sort data=sd1.have out=havsrt;                     |   x     5    15    76              */
    /* x        5       15     |   by id visit;                                          |   x     6    16    92              */
    /*                         | run;                                                    |   x     7    17   109              */
    /* options                 |                                                         |   x     8    18   127              */
    /*  validvarname=upcase;   | data want;                                              |   x     9    19   146              */
    /* libname sd1 "d:/sd1";   |   retain cumm 0;                                        |   x    10    20   166              */
    /* data sd1.have;          |   set havsrt;                                           |                                    */
    /* input id$ visit value;  |   by id ;                                               |                                    */
    /* cards4;                 |   if first.id then cumm = value;                        |                                    */
    /* n 6 06                  |   else cumm + value;                                    |                                    */
    /* n 7 07                  | run;                                                    |                                    */
    /* n 8 08                  |                                                         |                                    */
    /* n 9 09                  |----------------------------------------------------------------------------------------------*/
    /* n 10 10                 | 2 R SQL                                                 |                                    */
    /* n 1 01                  | =======                                                 |     ID VISIT VALUE CUMVAL          */
    /* n 1 01                  |                                                         |                                    */
    /* n 2 02                  | %utl_rbeginx;                                           |  1   n     1     1      1          */
    /* n 3 03                  | parmcards4;                                             |  2   n     1     1      2          */
    /* n 4 04                  | library(haven)                                          |  3   n     2     2      4          */
    /* n 5 05                  | library(sqldf)                                          |  4   n     3     3      7          */
    /* x 6 16                  | source("c:/oto/fn_tosas9x.r")                           |  5   n     4     4     11          */
    /* x 7 17                  | options(sqldf.dll = "d:/dll/sqlean.dll")                |  6   n     5     5     16          */
    /* x 8 18                  | have<-read_sas("d:/sd1/have.sas7bdat")                  |  7   n     6     6     22          */
    /* x 9 19                  | print(have)                                             |  8   n     7     7     29          */
    /* x 10 20                 | want<-sqldf('                                           |  9   n     8     8     37          */
    /* x 1 11                  | select                                                  |  10  n     9     9     46          */
    /* x 1 11                  |     id                                                  |  11  n    10    10     56          */
    /* x 2 12                  |    ,visit                                               |  12  x     1    11     11          */
    /* x 3 13                  |    ,value                                               |  13  x     1    11     22          */
    /* x 4 14                  |    ,sum(value) over (                                   |  14  x     2    12     34          */
    /* x 5 15                  |      partition by id                                    |  15  x     3    13     47          */
    /* ;;;;                    |      order by visit                                     |  16  x     4    14     61          */
    /* run;quit;               |      rows between                                       |  17  x     5    15     76          */
    /*                         |        unbounded preceding and current row              |  18  x     6    16     92          */
    /*                         |     ) as cumval                                         |  19  x     7    17    109          */
    /*                         | from                                                    |  20  x     8    18    127          */
    /*                         |     have                                                |  21  x     9    19    146          */
    /*                         | order                                                   |  22  x    10    20    166          */
    /*                         |     by id, visit                                        |                                    */
    /*                         |   ')                                                    |                                    */
    /*                         | want                                                    |                                    */
    /*                         | fn_tosas9x(                                             |                                    */
    /*                         |       inp    = want                                     |                                    */
    /*                         |      ,outlib ="d:/sd1/"                                 |                                    */
    /*                         |      ,outdsn ="want"                                    |                                    */
    /*                         |      )                                                  |                                    */
    /*                         | ;;;;                                                    |                                    */
    /*                         | %utl_rendx;                                             |                                    */
    /*                         |                                                         |                                    */
    /*                         |----------------------------------------------------------------------------------------------*/
    /*                         |                                                         |                                    */
    /*                         | 3 python sql (see excel below)                          |     ID  VISIT  VALUE  cumval       */
    /*                         | ===============================                         |  0   n    1.0    1.0     1.0       */
    /*                         |                                                         |  1   n    1.0    1.0     2.0       */
    /*                         | proc datasets lib=sd1 nolist nodetails;                 |  2   n    2.0    2.0     4.0       */
    /*                         |  delete pywant;                                         |  3   n    3.0    3.0     7.0       */
    /*                         | run;quit;                                               |  4   n    4.0    4.0    11.0       */
    /*                         |                                                         |  5   n    5.0    5.0    16.0       */
    /*                         | %utl_pybeginx;                                          |  6   n    6.0    6.0    22.0       */
    /*                         | parmcards4;                                             |  7   n    7.0    7.0    29.0       */
    /*                         | exec(open('c:/oto/fn_python.py').read());               |  8   n    8.0    8.0    37.0       */
    /*                         | have,meta=ps.read_sas7bdat('d:/sd1/have.sas7bdat');     |  9   n    9.0    9.0    46.0       */
    /*                         | want=pdsql('''                                          |  10  n   10.0   10.0    56.0       */
    /*                         |  select                                     \           |  11  x    1.0   11.0    11.0       */
    /*                         |      id                                     \           |  12  x    1.0   11.0    22.0       */
    /*                         |     ,visit                                  \           |  13  x    2.0   12.0    34.0       */
    /*                         |     ,value                                  \           |  14  x    3.0   13.0    47.0       */
    /*                         |     ,sum(value) over (                      \           |  15  x    4.0   14.0    61.0       */
    /*                         |       partition by id                       \           |  16  x    5.0   15.0    76.0       */
    /*                         |       order by visit                        \           |  17  x    6.0   16.0    92.0       */
    /*                         |       rows between                          \           |  18  x    7.0   17.0   109.0       */
    /*                         |         unbounded preceding and current row \           |  19  x    8.0   18.0   127.0       */
    /*                         |      ) as cumval                            \           |  20  x    9.0   19.0   146.0       */
    /*                         |  from                                       \           |  21  x   10.0   20.0   166.0       */
    /*                         |      have                                   \           |                                    */
    /*                         |  order                                      \           |                                    */
    /*                         |      by id, visit                                       |                                    */
    /*                         |  ''');                                                  |                                    */
    /*                         | print(want);                                            |                                    */
    /*                         | fn_tosas9x(want,outlib='d:/sd1/' \                      |                                    */
    /*                         |   ,outdsn='pywant',timeest=3);                          |                                    */
    /*                         | ;;;;                                                    |                                    */
    /*                         | %utl_pyendx;                                            |                                    */
    /*                         |                                                         |                                    */
    /*                         | proc print data=sd1.pywant;                             |                                    */
    /*                         | run;quit;                                               |                                    */
    /*                         |                                                         |                                    */
    /*                         |----------------------------------------------------------------------------------------------*/
    /*                         |                                                         |                                    */
    /*                         | 4 SAS SQL SOLUTION FAILS                                |  ID VISIT VALUE VALUE              */
    /*                         |   Only works when id x visit unique else catesian issue |                                    */
    /*                         |   ===================================================== |  n     1     1     4 * should be 1 */
    /*                         |                                                         |  n     1     1       * missing ob  */
    /*                         | proc sql;                                               |  n     2     2     4               */
    /*                         |   create                                                |  n     3     3     7               */
    /*                         |     table want as                                       |  n     4     4    11               */
    /*                         |   select                                                |  n     5     5    16               */
    /*                         |       s1.id                                             |  n     6     6    22               */
    /*                         |      ,s1.visit                                          |  n     7     7    29               */
    /*                         |      ,s1.value                                          |  n     8     8    37               */
    /*                         |      ,sum(s2.value) as cumulative_value                 |  n     9     9    46               */
    /*                         |   from                                                  |  n    10    10    56               */
    /*                         |       sd1.have s1 join sd1.have s2                      |  x     1    11    44               */
    /*                         |   on                                                    |  x     2    12    34               */
    /*                         |           s1.id = s2.id                                 |  x     3    13    47               */
    /*                         |       and s1.visit >= s2.visit                          |  x     4    14    61               */
    /*                         |   group                                                 |  x     5    15    76               */
    /*                         |      by s1.id                                           |  x     6    16    92               */
    /*                         |        ,s1.visit                                        |  x     7    17   109               */
    /*                         |        ,s1.value                                        |  x     8    18   127               */
    /*                         |   order                                                 |  x     9    19   146               */
    /*                         |      by  s1.id                                          |  x    10    20   166               */
    /*                         |         ,s1.visit                                       |                                    */
    /*                         | ;quit;                                                  |                                    */
    /*                         |                                                         |                                    */
    /*                         |----------------------------------------------------------------------------------------------*/
    /*                         |                                                         |                                    */
    /*                         | 5 SAS HASH                                              |    ID VISIT VALUE    CIM           */
    /*                         | ==========                                              |                                    */
    /*                         |                                                         | 1   n     1     1      1           */
    /*                         | data want;                                              | 2   n     1     1      2           */
    /*                         |   set sd1.have;                                         | 3   n     2     2      4           */
    /*                         |   by id notsorted;                                      | 4   n     3     3      7           */
    /*                         |   if _n_=1 then do;                                     | 5   n     4     4     11           */
    /*                         |     declare hash h                                      | 6   n     5     5     16           */
    /*                         |       (dataset:'sd1.have (obs=0)'                       | 7   n     6     6     22           */
    /*                         |         ,ordered:'a'                                    | 8   n     7     7     29           */
    /*                         |         ,multidata:'Y');                                | 9   n     8     8     37           */
    /*                         |       h.definekey('visit');                             | 10  n     9     9     46           */
    /*                         |       h.definedata(all:'Y');                            | 11  n    10    10     56           */
    /*                         |       h.definedone();                                   | 12  x     1    11     11           */
    /*                         |     declare hiter hi ('h');                             | 13  x     1    11     22           */
    /*                         |   end;                                                  | 14  x     2    12     34           */
    /*                         |   h.add();                                              | 15  x     3    13     47           */
    /*                         |   if last.id;                                           | 16  x     4    14     61           */
    /*                         |   do while (hi.next()=0);                               | 17  x     5    15     76           */
    /*                         |     cum=sum(cum,value);                                 | 18  x     6    16     92           */
    /*                         |     output;                                             | 19  x     7    17    109           */
    /*                         |   end;                                                  | 20  x     8    18    127           */
    /*                         |   h.clear();                                            | 21  x     9    19    146           */
    /*                         | run;                                                    | 22  x    10    20    166           */
    /*                         |                                                         |                                    */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options
     validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
    input id$ visit value;
    cards4;
    n 6 06
    n 7 07
    n 8 08
    n 9 09
    n 10 10
    n 1 01
    n 1 01
    n 2 02
    n 3 03
    n 4 04
    n 5 05
    x 6 16
    x 7 17
    x 8 18
    x 9 19
    x 10 20
    x 1 11
    x 1 11
    x 2 12
    x 3 13
    x 4 14
    x 5 15
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*   ID    VISIT    VALUE                                                                                                 */
    /*                                                                                                                        */
    /*   n        6        6                                                                                                  */
    /*   n        7        7                                                                                                  */
    /*   n        8        8                                                                                                  */
    /*   n        9        9                                                                                                  */
    /*   n       10       10                                                                                                  */
    /*   n        1        1                                                                                                  */
    /*   n        1        1                                                                                                  */
    /*   n        2        2                                                                                                  */
    /*   n        3        3                                                                                                  */
    /*   n        4        4                                                                                                  */
    /*   n        5        5                                                                                                  */
    /*   x        6       16                                                                                                  */
    /*   x        7       17                                                                                                  */
    /*   x        8       18                                                                                                  */
    /*   x        9       19                                                                                                  */
    /*   x       10       20                                                                                                  */
    /*   x        1       11                                                                                                  */
    /*   x        1       11                                                                                                  */
    /*   x        2       12                                                                                                  */
    /*   x        3       13                                                                                                  */
    /*   x        4       14                                                                                                  */
    /*   x        5       15                                                                                                  */
    /**************************************************************************************************************************/

    /*   _
    / | | |__   __ _ ___  ___   ___  __ _ ___
    | | | `_ \ / _` / __|/ _ \ / __|/ _` / __|
    | | | |_) | (_| \__ \  __/ \__ \ (_| \__ \
    |_| |_.__/ \__,_|___/\___| |___/\__,_|___/

    */

    proc sort data=sd1.have out=havsrt;
      by id visit;
    run;

    data want;
      retain cumm 0;
      set havsrt;
      by id ;
      if first.id then cumm = value;
      else cumm + value;
    run;

    /**************************************************************************************************************************/
    /*    ID    VISIT    VALUE CUMM                                                                                           */
    /*                                                                                                                        */
    /*    n        1        1     1                                                                                           */
    /*    n        1        1     2                                                                                           */
    /*    n        2        2     4                                                                                           */
    /*    n        3        3     7                                                                                           */
    /*    n        4        4    11                                                                                           */
    /*    n        5        5    16                                                                                           */
    /*    n        6        6    22                                                                                           */
    /*    n        7        7    29                                                                                           */
    /*    n        8        8    37                                                                                           */
    /*    n        9        9    46                                                                                           */
    /*    n       10       10    56                                                                                           */
    /*    x        1       11    11                                                                                           */
    /*    x        1       11    22                                                                                           */
    /*    x        2       12    34                                                                                           */
    /*    x        3       13    47                                                                                           */
    /*    x        4       14    61                                                                                           */
    /*    x        5       15    76                                                                                           */
    /*    x        6       16    92                                                                                           */
    /*    x        7       17   109                                                                                           */
    /*    x        8       18   127                                                                                           */
    /*    x        9       19   146                                                                                           */
    /*    x       10       20   166                                                                                           */
    /**************************************************************************************************************************/

    /*___                   _
    |___ \   _ __ ___  __ _| |
      __) | | `__/ __|/ _` | |
     / __/  | |  \__ \ (_| | |
    |_____| |_|  |___/\__, |_|
                         |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete want;
    run;quit;

    %utl_rbeginx;
    parmcards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.r")
    options(sqldf.dll = "d:/dll/sqlean.dll")
    have<-read_sas("d:/sd1/have.sas7bdat")
    print(have)
    want<-sqldf('
    select
        id
       ,visit
       ,value
       ,sum(value) over (
         partition by id
         order by visit
         rows between
           unbounded preceding and current row
        ) as cumval
    from
        have
    order
        by id, visit
      ')
    want
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /* > want                    |  SAS                                                                                       */
    /*    ID VISIT VALUE cumval  |  ROWNAMES    ID    VISIT    VALUE    CUMVAL                                                */
    /*                           |                                                                                            */
    /* 1   n     1     1      1  |      1       n        1        1         1                                                 */
    /* 2   n     1     1      2  |      2       n        1        1         2                                                 */
    /* 3   n     2     2      4  |      3       n        2        2         4                                                 */
    /* 4   n     3     3      7  |      4       n        3        3         7                                                 */
    /* 5   n     4     4     11  |      5       n        4        4        11                                                 */
    /* 6   n     5     5     16  |      6       n        5        5        16                                                 */
    /* 7   n     6     6     22  |      7       n        6        6        22                                                 */
    /* 8   n     7     7     29  |      8       n        7        7        29                                                 */
    /* 9   n     8     8     37  |      9       n        8        8        37                                                 */
    /* 10  n     9     9     46  |     10       n        9        9        46                                                 */
    /* 11  n    10    10     56  |     11       n       10       10        56                                                 */
    /* 12  x     1    11     11  |     12       x        1       11        11                                                 */
    /* 13  x     1    11     22  |     13       x        1       11        22                                                 */
    /* 14  x     2    12     34  |     14       x        2       12        34                                                 */
    /* 15  x     3    13     47  |     15       x        3       13        47                                                 */
    /* 16  x     4    14     61  |     16       x        4       14        61                                                 */
    /* 17  x     5    15     76  |     17       x        5       15        76                                                 */
    /* 18  x     6    16     92  |     18       x        6       16        92                                                 */
    /* 19  x     7    17    109  |     19       x        7       17       109                                                 */
    /* 20  x     8    18    127  |     20       x        8       18       127                                                 */
    /* 21  x     9    19    146  |     21       x        9       19       146                                                 */
    /* 22  x    10    20    166  |     22       x       10       20       166                                                 */
    /**************************************************************************************************************************/

    /*____               _   _                             _
    |___ /   _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
      |_ \  | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
     ___) | | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
    |____/  | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
            |_|    |___/                                |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete pywant;
    run;quit;

    %utl_pybeginx;
    parmcards4;
    exec(open('c:/oto/fn_python.py').read());
    have,meta=ps.read_sas7bdat('d:/sd1/have.sas7bdat');
    want=pdsql('''
     select                                     \
         id                                     \
        ,visit                                  \
        ,value                                  \
        ,sum(value) over (                      \
          partition by id                       \
          order by visit                        \
          rows between                          \
            unbounded preceding and current row \
         ) as cumval                            \
     from                                       \
         have                                   \
     order                                      \
         by id, visit
     ''');
    print(want);
    fn_tosas9x(want,outlib='d:/sd1/' \
      ,outdsn='pywant',timeest=3);
    ;;;;
    %utl_pyendx;

    proc print data=sd1.pywant;
    run;quit;

    /**************************************************************************************************************************/
    /* PYTHON                       |    SAS                                                                                  */
    /*    ID  VISIT  VALUE  cumval  |    ID    VISIT    VALUE    CUMVAL                                                       */
    /*                              |                                                                                         */
    /* 0   n    1.0    1.0     1.0  |    n        1        1         1                                                        */
    /* 1   n    1.0    1.0     2.0  |    n        1        1         2                                                        */
    /* 2   n    2.0    2.0     4.0  |    n        2        2         4                                                        */
    /* 3   n    3.0    3.0     7.0  |    n        3        3         7                                                        */
    /* 4   n    4.0    4.0    11.0  |    n        4        4        11                                                        */
    /* 5   n    5.0    5.0    16.0  |    n        5        5        16                                                        */
    /* 6   n    6.0    6.0    22.0  |    n        6        6        22                                                        */
    /* 7   n    7.0    7.0    29.0  |    n        7        7        29                                                        */
    /* 8   n    8.0    8.0    37.0  |    n        8        8        37                                                        */
    /* 9   n    9.0    9.0    46.0  |    n        9        9        46                                                        */
    /* 10  n   10.0   10.0    56.0  |    n       10       10        56                                                        */
    /* 11  x    1.0   11.0    11.0  |    x        1       11        11                                                        */
    /* 12  x    1.0   11.0    22.0  |    x        1       11        22                                                        */
    /* 13  x    2.0   12.0    34.0  |    x        2       12        34                                                        */
    /* 14  x    3.0   13.0    47.0  |    x        3       13        47                                                        */
    /* 15  x    4.0   14.0    61.0  |    x        4       14        61                                                        */
    /* 16  x    5.0   15.0    76.0  |    x        5       15        76                                                        */
    /* 17  x    6.0   16.0    92.0  |    x        6       16        92                                                        */
    /* 18  x    7.0   17.0   109.0  |    x        7       17       109                                                        */
    /* 19  x    8.0   18.0   127.0  |    x        8       18       127                                                        */
    /* 20  x    9.0   19.0   146.0  |    x        9       19       146                                                        */
    /* 21  x   10.0   20.0   166.0  |    x       10       20       166                                                        */
    /**************************************************************************************************************************/

    /*  _                               _    __       _ _
    | || |    ___  __ _ ___   ___  __ _| |  / _| __ _(_) |___
    | || |_  / __|/ _` / __| / __|/ _` | | | |_ / _` | | / __|
    |__   _| \__ \ (_| \__ \ \__ \ (_| | | |  _| (_| | | \__ \
       |_|   |___/\__,_|___/ |___/\__, |_| |_|  \__,_|_|_|___/
                                     |_|
    */

    proc sql;
      create
        table want as
      select
          s1.id
         ,s1.visit
         ,s1.value
         ,sum(s2.value) as cumval
      from
          sd1.have s1 join sd1.have s2
      on
              s1.id = s2.id
          and s1.visit >= s2.visit
      group
         by s1.id
           ,s1.visit
           ,s1.value
      order
         by  s1.id
            ,s1.visit
    ;quit;

    /**************************************************************************************************************************/
    /*  ID    VISIT    VALUE    CUMVAL                                                                                        */
    /*                                                                                                                        */
    /*  n        1        1         4 ** wrong                                                                                */
    /*  n        2        2         4                                                                                         */
    /*  n        3        3         7                                                                                         */
    /*  n        4        4        11                                                                                         */
    /*  n        5        5        16                                                                                         */
    /*  n        6        6        22                                                                                         */
    /*  n        7        7        29                                                                                         */
    /*  n        8        8        37                                                                                         */
    /*  n        9        9        46                                                                                         */
    /*  n       10       10        56                                                                                         */
    /*  x        1       11        44                                                                                         */
    /*  x        2       12        34                                                                                         */
    /*  x        3       13        47                                                                                         */
    /*  x        4       14        61                                                                                         */
    /*  x        5       15        76                                                                                         */
    /*  x        6       16        92                                                                                         */
    /*  x        7       17       109                                                                                         */
    /*  x        8       18       127                                                                                         */
    /*  x        9       19       146                                                                                         */
    /*  x       10       20       166                                                                                         */
    /**************************************************************************************************************************/

    /*  _                     _               _
    | || |    ___  __ _ ___  | |__   __ _ ___| |__
    | || |_  / __|/ _` / __| | `_ \ / _` / __| `_ \
    |__   _| \__ \ (_| \__ \ | | | | (_| \__ \ | | |
       |_|   |___/\__,_|___/ |_| |_|\__,_|___/_| |_|

    */

    data want;
      set sd1.have;
      by id notsorted;
      if _n_=1 then do;
        declare hash h
          (dataset:'sd1.have (obs=0)'
            ,ordered:'a'
            ,multidata:'Y');
          h.definekey('visit');
          h.definedata(all:'Y');
          h.definedone();
        declare hiter hi ('h');
      end;
      h.add();
      if last.id;
      do while (hi.next()=0);
        cum=sum(cum,value);
        output;
      end;
      h.clear();
    run;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

