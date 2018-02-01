# utl_long_to_fat_transposing_multiple_variables_by_groups_with_unique_ids_within_group
Transposing multiple variables by groups with unique ids within group. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    Long to Fat - Transposing multiple variables by groups with unique ids within group.
    proc transpose requires you to sort and do n transpose one per variable and then merge.

       github
       https://goo.gl/ms9NmP
       https://github.com/rogerjdeangelis/utl_long_to_fat_transposing_multiple_variables_by_groups_with_unique_ids_within_group

       SOLUTIONS (No tabulate solutions because you cannot create a fat dataset out of tabulate.
                  Tabulate boes not honor ODS output (use proc report?). untranspose fat to skinny )

            1. OPS solution
               (clear code) sort-transpose-transpose-merge
            2. One proc report and a simple rename macro
                  I think this is the most powerfull because you can add summary statistics.
                  Would be simpler but 'proc report' does not honor 'ods output'
            3. Gather-sort-transpose
               (gather output provides a very powerfull 'normalized' output data structure)
               It has the ability to store numeric and characater data in one column and adds
               the meta data you need to recreate the original char and num variables.
               (Like Oracle Clinical Center fact table)
               (Alea Iacta https://github.com/clindocu)
            4. Gather-fix-sort=transpose
               I added this because operating on the output of gather has many applications
            5. Gather-report
            6. proc summary idgroup - close but does not solve the problem
               provides an interesting data structure
               Like report suppots many additional summary functions. Multithreaded
            7. Sort-sql-merge Søren Lassen 000002b7c5cf1459-dmarc-request@listserv.uga.edu

      HAVE
      ====
          Up to 40 obs WORK.HAVE total obs=15

                  CUSTOMER_                   CHAR_    CHAR_     CHAR_
          Obs        ID        ACCOUNT_ID    NUMBER    VALUE    POINTS

            1    1111111111    8888888888     1234       14       102
            2    1111111111    8888888888     4567        .        96
            3    1111111111    8888888888     7890        0        96
            4    1111111111    8888888888     1000       47        98
            5    1111111111    8888888888     1001       42       110
            6    1111111111    8888888888     1002        0       134
            7    2222222222    9999999999     5555        0        91
            8    2222222222    9999999999     6666      102        92
            9    2222222222    9999999999     1234       33        95
           10    2222222222    9999999999     4567        0        91
           11    2222222222    9999999999     7890        0        98
           12    2222222222    9999999999     1001        7        92
           13    2222222222    9999999999     1000       41        92
           14    2222222222    4444444444     3333       42         .
           15    2222222222    4444444444     1234        0       134

      WANT
      ====
           Middle Observation(1 ) of wide3 - Total Obs 3

            Variables
            -- NUMERIC --             VALUES

           CUSTOMER_ID       N8       1111111111
           ACCOUNT_ID        N8       8888888888

           CHAR_VALUE1234    N8       14
           CHAR_VALUE4567    N8       .
           CHAR_VALUE7890    N8       0
           CHAR_VALUE1000    N8       47
           CHAR_VALUE1001    N8       42
           CHAR_VALUE1002    N8       0
           CHAR_VALUE3333    N8       .
           CHAR_VALUE5555    N8       .
           CHAR_VALUE6666    N8       .

           CHAR_POINTS1234   N8       102
           CHAR_POINTS4567   N8       96
           CHAR_POINTS7890   N8       96
           CHAR_POINTS1000   N8       98
           CHAR_POINTS1001   N8       110
           CHAR_POINTS1002   N8       134
           CHAR_POINTS3333   N8       .
           CHAR_POINTS5555   N8       .
           CHAR_POINTS6666   N8       .

    *                            _       _   _
      ___  _ __  ___   ___  ___ | |_   _| |_(_) ___  _ __
     / _ \| '_ \/ __| / __|/ _ \| | | | | __| |/ _ \| '_ \
    | (_) | |_) \__ \ \__ \ (_) | | |_| | |_| | (_) | | | |
     \___/| .__/|___/ |___/\___/|_|\__,_|\__|_|\___/|_| |_|
          |_|
    ;

    neilfrnnd@gmail.com

    data have;
    retain cnt;
    input customer_ID ACCOUNT_ID char_number char_VALUE   char_points;
    cards4;
    1111111111 8888888888 1234 14 102
    1111111111 8888888888 4567 . 96
    1111111111 8888888888 7890 0 96
    1111111111 8888888888 1000 47 98
    1111111111 8888888888 1001 42 110
    1111111111 8888888888 1002 0 134
    2222222222 9999999999 5555 0 91
    2222222222 9999999999 6666 102 92
    2222222222 9999999999 1234 33 95
    2222222222 9999999999 4567 0 91
    2222222222 9999999999 7890 0 98
    2222222222 9999999999 1001 7 92
    2222222222 9999999999 1000 41 92
    2222222222 4444444444 3333 42 .
    2222222222 4444444444 1234 0 134
    ;;;;
    run;quit;


    /*Sorting prior to transposing */
    proc sort data= have  out=sorted_table_1 ;
      by Customer_ID ACCOUNT_ID;
    run;

    /*Transposing two variables*/
    proc transpose data=sorted_table_1  out=wide1 prefix=char_VALUE;
        by Customer_ID ACCOUNT_ID ;
        id char_number;
        var char_value ;
    run;


    proc transpose data=sorted_table_1  out=wide2 prefix=char_POINTS;
        by Customer_ID ACCOUNT_ID ;
        id char_number;
        var char_points;
    run;

    /*The multiple transposed data files then are merged back.  */
    data wide3;
        merge       wide1(drop=_name_)
                            wide2(drop=_name_);
        by Customer_ID ACCOUNT_ID;
    run;

    *                                             _
      ___  _ __   ___   _ __ ___ _ __   ___  _ __| |_
     / _ \| '_ \ / _ \ | '__/ _ \ '_ \ / _ \| '__| __|
    | (_) | | | |  __/ | | |  __/ |_) | (_) | |  | |_
     \___/|_| |_|\___| |_|  \___| .__/ \___/|_|   \__|
                                |_|
    ;

    * one report with a simple rename macro;

    * unfortunately 'proc report' does not honor 'ods output' or 'out=';
    * proc report is an amazing tool to restructure data and its the 'data structure stupid"
    * n ot the static report;

    data have;
    retain cnt;
    input customer_ID ACCOUNT_ID char_number char_VALUE   char_points;
    cards4;
    1111111111 8888888888 1234 14 102
    1111111111 8888888888 4567 . 96
    1111111111 8888888888 7890 0 96
    1111111111 8888888888 1000 47 98
    1111111111 8888888888 1001 42 110
    1111111111 8888888888 1002 0 134
    2222222222 9999999999 5555 0 91
    2222222222 9999999999 6666 102 92
    2222222222 9999999999 1234 33 95
    2222222222 9999999999 4567 0 91
    2222222222 9999999999 7890 0 98
    2222222222 9999999999 1001 7 92
    2222222222 9999999999 1000 41 92
    2222222222 4444444444 3333 42 .
    2222222222 4444444444 1234 0 134
    ;;;;
    run;quit;


    * you need the macro below to rename columns;
    proc report data=work.have ls=171 ps=65  split="/"
      nocenter missing out=havFin (rename=(%utl_rptRenTwo(have,char_number,char_points,char_value)));
    column  customer_id account_id char_number, (char_value char_points);
    define  customer_id / group format= 13.      spacing=2   right "customer_id" ;
    define  account_id / group format= 13.     spacing=2   right "account_id" ;
    define  char_number / across    spacing=2   left "char_number" ;
    define  char_points /  format= 13.     spacing=2   right "char_points" ;
    define  char_value /  format= 8.     spacing=2   right "char_value" ;
    run;quit;

    %macro utl_rptRenTwo(nrm,acx,var1,var2);
    %let nams=;
    %let rc=
     %sysfunc(dosubl('
       %symdel nams / nowarn;
       proc sql ;
         select cats("_C",put(monotonic()+2,5.),"_ =_",lon) into :nams separated by " "
         from
          (select distinct cats("&var1._",&acx) as lon  from &nrm
              union all
           select distinct cats("&var2._",&acx) as lon from &nrm )
       ;quit;
       '));
       &nams
    %mend utl_rptRenTwo;


    p to 40 obs from havFin total obs=3

                               _CHAR_  _CHAR_  _CHAR_  _CHAR_  _CHAR_  _CHAR_  _CHAR_  _CHAR_
         CUSTOMER_            POINTS_ POINTS_ POINTS_ POINTS_ POINTS_ POINTS_ POINTS_ POINTS_
    Obs     ID     ACCOUNT_ID   1000    1001    1002    1234    3333    4567    5555    6666

     1  1111111111 8888888888    47      98      42     110      0      134      14     102
     2  2222222222 4444444444     .       .       .       .      .        .       0     134
     3  2222222222 9999999999    41      92       7      92      .        .      33      95

     _CHAR_ _CHAR_ _CHAR_ _CHAR_ _CHAR_ _CHAR_ _CHAR_ _CHAR_ _CHAR_ _CHAR_
    POINTS_ VALUE_ VALUE_ VALUE_ VALUE_ VALUE_ VALUE_ VALUE_ VALUE_ VALUE_
      7890   1000   1001   1002   1234   3333   4567   5555   6666   7890

        .      .      .     96      .      .       .     .      0     96
       42      .      .      .      .      .       .     .      .      .
        .      .      0     91      0     91     102    92      0     98


    Middle Observation(1 ) of havFin - Total Obs 3

     -- NUMERIC --
    CUSTOMER_ID          N8       1111111111
    ACCOUNT_ID           N8       8888888888

    _CHAR_POINTS_1000    N8       47
    _CHAR_POINTS_1001    N8       98
    _CHAR_POINTS_1002    N8       42
    _CHAR_POINTS_1234    N8       110
    _CHAR_POINTS_3333    N8       0
    _CHAR_POINTS_4567    N8       134
    _CHAR_POINTS_5555    N8       14
    _CHAR_POINTS_6666    N8       102
    _CHAR_POINTS_7890    N8       .

    _CHAR_VALUE_1000     N8       .
    _CHAR_VALUE_1001     N8       .
    _CHAR_VALUE_1002     N8       96
    _CHAR_VALUE_1234     N8       .
    _CHAR_VALUE_3333     N8       .
    _CHAR_VALUE_4567     N8       .
    _CHAR_VALUE_5555     N8       .
    _CHAR_VALUE_6666     N8       0
    _CHAR_VALUE_7890     N8       96


    *            _   _                 _
      __ _  __ _| |_| |__   ___ _ __  | |_ _ __ __ _ _ __
     / _` |/ _` | __| '_ \ / _ \ '__| | __| '__/ _` | '_ \
    | (_| | (_| | |_| | | |  __/ |    | |_| | | (_| | | | |
     \__, |\__,_|\__|_| |_|\___|_|     \__|_|  \__,_|_| |_|
     |___/
    ;

    This eliminates the havFix dataset

      Three steps

       1.  gather
       2.  sort
       3.  proc transpose


    Here is the utl_gather solution
    (if I have time I will add 4 more (including array/untranspose?)

    Good question and I think this is important.

    INPUT
    =====

     WORK.HAVE total obs=15

        CUSTOMER_                   CHAR_    CHAR_     CHAR_
           ID        ACCOUNT_ID    NUMBER    VALUE    POINTS

       1111111111    8888888888     1234       14       102
       1111111111    8888888888     4567        .        96
       1111111111    8888888888     7890        0        96
       1111111111    8888888888     1000       47        98
       1111111111    8888888888     1001       42       110
       1111111111    8888888888     1002        0       134
       2222222222    9999999999     5555        0        91
       2222222222    9999999999     6666      102        92
       2222222222    9999999999     1234       33        95
       2222222222    9999999999     4567        0        91
       2222222222    9999999999     7890        0        98
       2222222222    9999999999     1001        7        92
       2222222222    9999999999     1000       41        92
       2222222222    4444444444     3333       42         .
       2222222222    4444444444     1234        0       134


    PROCESS
    =======

    * note proc transpose cannot do this - check out 'untranspose Rick and Art)
    %utl_gather(have,var,vals,customer_id account_id char_number,havxpo,withformats=y);

    proc sort data=havXpo out=havSrt;
      by customer_id account_id var;
    run;quit;

    proc transpose data=havSrt out=havXpoXpo(drop=_name_);;
      by customer_id account_id;
      id var char_number;
      var vals;
    run;quit;

    *            _   _                  __ _
      __ _  __ _| |_| |__   ___ _ __   / _(_)_  __ __  ___ __   ___
     / _` |/ _` | __| '_ \ / _ \ '__| | |_| \ \/ / \ \/ / '_ \ / _ \
    | (_| | (_| | |_| | | |  __/ |    |  _| |>  <   >  <| |_) | (_) |
     \__, |\__,_|\__|_| |_|\___|_|    |_| |_/_/\_\ /_/\_\ .__/ \___/
     |___/                                              |_|
    ;

    Here is the utl_gather solution
    (if I have time I will add 4 more (including array/untranspose?)

    Good question and I think this is important.

    INPUT
    =====

     WORK.HAVE total obs=15

        CUSTOMER_                   CHAR_    CHAR_     CHAR_
           ID        ACCOUNT_ID    NUMBER    VALUE    POINTS

       1111111111    8888888888     1234       14       102
       1111111111    8888888888     4567        .        96
       1111111111    8888888888     7890        0        96
       1111111111    8888888888     1000       47        98
       1111111111    8888888888     1001       42       110
       1111111111    8888888888     1002        0       134
       2222222222    9999999999     5555        0        91
       2222222222    9999999999     6666      102        92
       2222222222    9999999999     1234       33        95
       2222222222    9999999999     4567        0        91
       2222222222    9999999999     7890        0        98
       2222222222    9999999999     1001        7        92
       2222222222    9999999999     1000       41        92
       2222222222    4444444444     3333       42         .
       2222222222    4444444444     1234        0       134


    PROCESS
    =======

    * note proc transpose cannot do this - check out 'untranspose Rick and Art)
    %utl_gather(have,var,vals,customer_id account_id char_number,havxpo,withformats=y);


    /* gather output is a very powerfull data structure

     WORK.HAVXPO total obs=30
                                                            Can be       Allows both
         CUSTOMER_                   CHAR_                chr plus num  Char and Numeric
            ID        ACCOUNT_ID    NUMBER        VAR        VALS    _COLFORMAT    _COLTYP

        1111111111    8888888888     1234     CHAR_VALUE       14     BEST12.         N
        1111111111    8888888888     1234     CHAR_POINTS     102     BEST12.         N
        1111111111    8888888888     4567     CHAR_VALUE        .     BEST12.         N
        1111111111    8888888888     4567     CHAR_POINTS      96     BEST12.         N
        1111111111    8888888888     7890     CHAR_VALUE        0     BEST12.         N
        1111111111    8888888888     7890     CHAR_POINTS      96     BEST12.         N
        1111111111    8888888888     1000     CHAR_VALUE       47     BEST12.         N
        1111111111    8888888888     1000     CHAR_POINTS      98     BEST12.         N
        1111111111    8888888888     1001     CHAR_VALUE       42     BEST12.         N
        1111111111    8888888888     1001     CHAR_POINTS     110     BEST12.         N
        1111111111    8888888888     1002     CHAR_VALUE        0     BEST12.         N
        1111111111    8888888888     1002     CHAR_POINTS     134     BEST12.         N
        2222222222    9999999999     5555     CHAR_VALUE        0     BEST12.         N
        2222222222    9999999999     5555     CHAR_POINTS      91     BEST12.         N
        2222222222    9999999999     6666     CHAR_VALUE      102     BEST12.         N
        2222222222    9999999999     6666     CHAR_POINTS      92     BEST12.         N
        2222222222    9999999999     1234     CHAR_VALUE       33     BEST12.         N
        2222222222    9999999999     1234     CHAR_POINTS      95     BEST12.         N
        2222222222    9999999999     4567     CHAR_VALUE        0     BEST12.         N
        2222222222    9999999999     4567     CHAR_POINTS      91     BEST12.         N
        2222222222    9999999999     7890     CHAR_VALUE        0     BEST12.         N
        2222222222    9999999999     7890     CHAR_POINTS      98     BEST12.         N
        2222222222    9999999999     1001     CHAR_VALUE        7     BEST12.         N
        2222222222    9999999999     1001     CHAR_POINTS      92     BEST12.         N
        2222222222    9999999999     1000     CHAR_VALUE       41     BEST12.         N
        2222222222    9999999999     1000     CHAR_POINTS      92     BEST12.         N
        2222222222    4444444444     3333     CHAR_VALUE       42     BEST12.         N
        2222222222    4444444444     3333     CHAR_POINTS       .     BEST12.         N
        2222222222    4444444444     1234     CHAR_VALUE        0     BEST12.         N
        2222222222    4444444444     1234     CHAR_POINTS     134     BEST12.         N
    */

    * use char_number as a suffix;
    data havFix/view=havFix;
      set havXpo;
      var=cats(scan(var,2,'_'),put(char_number,4.));
    run;quit;

    proc sort data=havFix out=havSrt;
      by customer_id account_id var;
    run;quit;

    proc transpose data=havSrt out=havXpoXpo(drop=_name_);;
      by customer_id account_id;
      id var;
      var vals;
    run;quit;


    VERIFICATION
    =============

            MY OUTPUT                  YOUR OUTPUT
            =========                  ===========

    CUSTOMER_ID   N8   1111111111   CUSTOMER_ID         1111111111
    ACCOUNT_ID    N8   8888888888   ACCOUNT_ID          8888888888
    POINTS1000    N8   98           CHAR_VALUE1234      14  checks
    POINTS1001    N8   110          CHAR_VALUE4567      .
    POINTS1002    N8   134          CHAR_VALUE7890      0   checks
    POINTS1234    N8   102          CHAR_VALUE1000      47  checks
    POINTS4567    N8   96           CHAR_VALUE1001      42  checks
    POINTS7890    N8   96           CHAR_VALUE1002      0   checks
    VALUE1000     N8   47           CHAR_VALUE3333      .
    VALUE1001     N8   42           CHAR_VALUE5555      .
    VALUE1002     N8   0            CHAR_VALUE6666      .
    VALUE1234     N8   14           CHAR_POINTS1234     102 checks
    VALUE4567     N8   .            CHAR_POINTS4567     96  checks
    VALUE7890     N8   0            CHAR_POINTS7890     96  checks
    POINTS3333    N8   .            CHAR_POINTS1000     98  checks
    VALUE3333     N8   .            CHAR_POINTS1001     110 checks
    POINTS5555    N8   .            CHAR_POINTS1002     134 checks
    POINTS6666    N8   .            CHAR_POINTS3333     .
    VALUE5555     N8   .            CHAR_POINTS5555     .
    VALUE6666     N8   .            CHAR_POINTS6666     .

    *            _   _                            _
      __ _  __ _| |_| |__   ___ _ __   _ __ _ __ | |_
     / _` |/ _` | __| '_ \ / _ \ '__| | '__| '_ \| __|
    | (_| | (_| | |_| | | |  __/ |    | |  | |_) | |_
     \__, |\__,_|\__|_| |_|\___|_|    |_|  | .__/ \__|
     |___/                                 |_|
    ;


    * note proc transpose cannot do this - check out 'untranspose Rick and Art);

    %utl_gather(have,var,vals,customer_id account_id char_number,havxpo,valformat=8.,withformats=y);

    /* gather output is a very powerfull data structure

     WORK.HAVXPO total obs=30
                                                            Can be       Allows both
         CUSTOMER_                   CHAR_                chr and num   Char and Numeric
            ID        ACCOUNT_ID    NUMBER        VAR        VALS    _COLFORMAT    _COLTYP

        1111111111    8888888888     1234     CHAR_VALUE       14     BEST12.         N
        1111111111    8888888888     1234     CHAR_POINTS     102     BEST12.         N
        1111111111    8888888888     4567     CHAR_VALUE        .     BEST12.         N
        1111111111    8888888888     4567     CHAR_POINTS      96     BEST12.         N
        1111111111    8888888888     7890     CHAR_VALUE        0     BEST12.         N
        1111111111    8888888888     7890     CHAR_POINTS      96     BEST12.         N
        1111111111    8888888888     1000     CHAR_VALUE       47     BEST12.         N
        1111111111    8888888888     1000     CHAR_POINTS      98     BEST12.         N
        1111111111    8888888888     1001     CHAR_VALUE       42     BEST12.         N
        1111111111    8888888888     1001     CHAR_POINTS     110     BEST12.         N
        1111111111    8888888888     1002     CHAR_VALUE        0     BEST12.         N
        1111111111    8888888888     1002     CHAR_POINTS     134     BEST12.         N
        2222222222    9999999999     5555     CHAR_VALUE        0     BEST12.         N
        2222222222    9999999999     5555     CHAR_POINTS      91     BEST12.         N
        2222222222    9999999999     6666     CHAR_VALUE      102     BEST12.         N
        2222222222    9999999999     6666     CHAR_POINTS      92     BEST12.         N
        2222222222    9999999999     1234     CHAR_VALUE       33     BEST12.         N
        2222222222    9999999999     1234     CHAR_POINTS      95     BEST12.         N
        2222222222    9999999999     4567     CHAR_VALUE        0     BEST12.         N
        2222222222    9999999999     4567     CHAR_POINTS      91     BEST12.         N
        2222222222    9999999999     7890     CHAR_VALUE        0     BEST12.         N
        2222222222    9999999999     7890     CHAR_POINTS      98     BEST12.         N
        2222222222    9999999999     1001     CHAR_VALUE        7     BEST12.         N
        2222222222    9999999999     1001     CHAR_POINTS      92     BEST12.         N
        2222222222    9999999999     1000     CHAR_VALUE       41     BEST12.         N
        2222222222    9999999999     1000     CHAR_POINTS      92     BEST12.         N
        2222222222    4444444444     3333     CHAR_VALUE       42     BEST12.         N
        2222222222    4444444444     3333     CHAR_POINTS       .     BEST12.         N
        2222222222    4444444444     1234     CHAR_VALUE        0     BEST12.         N
        2222222222    4444444444     1234     CHAR_POINTS     134     BEST12.         N
    */

    * use char_number as a suffix;
    data havFix;
      set havXpo;
      var=cats(scan(var,2,'_'),put(char_number,4.));
    run;quit;

    * you need the macro below to rename columns;
    proc report data=work.havfix ls=171 ps=65  split="/"
      nocenter missing out=havFin(rename=(%utl_rptRen(havFix,var)));
    column  customer_id account_id var, vals;
    define  customer_id / group format= 13.      spacing=2   right "customer_id" ;
    define  account_id / group format= 13.     spacing=2   right "account_id" ;
    define  var / across format= $22.    spacing=2   left "var" ;
    define  vals / sum format= 8.     spacing=2   right "vals" ;
    run;quit;

    %macro utl_rptRen(nrm,acx);
    %let nams=;
    %let rc=
     %sysfunc(dosubl('
       %symdel nams / nowarn;
       proc sql noprint;
         select cats("_C",put(monotonic()+1,5.),"_ =_",lon) into :nams separated by " "
         from
          (select distinct &acx as lon  from &nrm)
       ;quit;
       '));
       &nams
    %mend utl_rptRen;

    *
     ___ _   _ _ __ ___  _ __ ___   __ _ _ __ _   _
    / __| | | | '_ ` _ \| '_ ` _ \ / _` | '__| | | |
    \__ \ |_| | | | | | | | | | | | (_| | |  | |_| |
    |___/\__,_|_| |_| |_|_| |_| |_|\__,_|_|   \__, |
                                              |___/
     _     _
    (_) __| | __ _ _ __ ___  _   _ _ __
    | |/ _` |/ _` | '__/ _ \| | | | '_ \
    | | (_| | (_| | | | (_) | |_| | |_) |
    |_|\__,_|\__, |_|  \___/ \__,_| .__/
             |___/                |_|
    ;

    data have;
    retain cnt;
    input customer_ID ACCOUNT_ID char_number char_VALUE   char_points;
    cards4;
    1111111111 8888888888 1234 14 102
    1111111111 8888888888 4567 . 96
    1111111111 8888888888 7890 0 96
    1111111111 8888888888 1000 47 98
    1111111111 8888888888 1001 42 110
    1111111111 8888888888 1002 0 134
    2222222222 9999999999 5555 0 91
    2222222222 9999999999 6666 102 92
    2222222222 9999999999 1234 33 95
    2222222222 9999999999 4567 0 91
    2222222222 9999999999 7890 0 98
    2222222222 9999999999 1001 7 92
    2222222222 9999999999 1000 41 92
    2222222222 4444444444 3333 42 .
    2222222222 4444444444 1234 0 134
    ;;;;
    run;quit;


    * you can almost do it with a simple proc summary
    * the problem lies with both renamining and chaging the
      position of transposed variable.
    * proc summary cannot use char_number for variable names and
    * variables are laid out in the order of the original data

    * see the output below from this simple proc summary;

    proc summary data=have nway;
      by  customer_id account_id notsorted;
      output out=want (drop=_type_ rename=_freq_=count)
           idgroup(out[7] (char_value char_points  char_number)=);
    run;quit;

    The problem is that the char_value and char_points zre laid
    out in the order of char_number.

    For record 2
        char_value_1 should have name char_value_1234
    For record
        char_value_1 should have name char_value_5555

    So we have to move char_value_1 to another colum


    WORK.WANT  total obs=3

               CUSTOMER_
       Obs        ID        ACCOUNT_ID    COUNT

        1     1111111111    8888888888      6
        2     2222222222    9999999999      7
        3     2222222222    4444444444      2


               CHAR_       CHAR_       CHAR_       CHAR_       CHAR_       CHAR_       CHAR_
       Obs   NUMBER_1    NUMBER_2    NUMBER_3    NUMBER_4    NUMBER_5    NUMBER_6    NUMBER_7

        1      1234        4567        7890        1000        1001        1002           .
        2      5555        6666        1234        4567        7890        1001        1000
        3      3333        1234           .           .           .           .           .

              CHAR_      CHAR_      CHAR_      CHAR_      CHAR_      CHAR_      CHAR_
       Obs   VALUE_1    VALUE_2    VALUE_3    VALUE_4    VALUE_5    VALUE_6    VALUE_7

        1       14          .          0         47         42         0           .
        2        0        102         33          0          0         7          41
        3       42          0          .          .          .         .           .


               CHAR_       CHAR_       CHAR_             CHAR_       CHAR_       CHAR_
       Obs   POINTS_1    POINTS_2    POINTS_3   POINTS_4    POINTS_5    POINTS_6    POINTS_7

        1       102          96         96         98          110         134          .
        2        91          92         95         91           98          92         92
        3         .         134          .          .            .           .          .


    *                                     _
     ___  ___  _ __ ___ _ __    ___  __ _| |
    / __|/ _ \| '__/ _ \ '_ \  / __|/ _` | |
    \__ \ (_) | | |  __/ | | | \__ \ (_| | |
    |___/\___/|_|  \___|_| |_| |___/\__, |_|
                                       |_|
    ;

    Søren Lassen
    Søren Lassen's profile photo
    000002b7c5cf1459-dmarc-request@listserv.uga.edu

    data have;
    retain cnt;
    input customer_ID ACCOUNT_ID char_number char_VALUE   char_points;
    cards4;
    1111111111 8888888888 1234 14 102
    1111111111 8888888888 4567 . 96
    1111111111 8888888888 7890 0 96
    1111111111 8888888888 1000 47 98
    1111111111 8888888888 1001 42 110
    1111111111 8888888888 1002 0 134
    2222222222 9999999999 5555 0 91
    2222222222 9999999999 6666 102 92
    2222222222 9999999999 1234 33 95
    2222222222 9999999999 4567 0 91
    2222222222 9999999999 7890 0 98
    2222222222 9999999999 1001 7 92
    2222222222 9999999999 1000 41 92
    2222222222 4444444444 3333 42 .
    2222222222 4444444444 1234 0 134
    ;;;;
    run;quit;


    I added the macro variable contents ;

    * these solutions are useful especially as an academic/practical exercise and help users understand data structures;

    proc sort data= have  out=sorted_table_1 ;
      by Customer_ID ACCOUNT_ID;
    run;


    proc sql noprint;
      select distinct
        catt('sorted_table_1(where=(char_number=',char_number,
             ') rename=(char_value=char_value',char_number,' char_points=char_points',char_number,'))')
        into :indata separated by ' '
        from have
        ;
    quit;

    /* macro variable &=indata
    sorted_table_1(where=(char_number=1000) rename=(char_value=char_value1000 char_points=char_points1000))
    sorted_table_1(where=(char_number=1001) rename=(char_value=char_value1001 char_points=char_points1001))
    sorted_table_1(where=(char_number=1002) rename=(char_value=char_value1002 char_points=char_points1002))
    sorted_table_1(where=(char_number=1234) rename=(char_value=char_value1234 char_points=char_points1234))
    sorted_table_1(where=(char_number=3333) rename=(char_value=char_value3333 char_points=char_points3333))
    sorted_table_1(where=(char_number=4567) rename=(char_value=char_value4567 char_points=char_points4567))
    sorted_table_1(where=(char_number=5555) rename=(char_value=char_value5555 char_points=char_points5555))
    sorted_table_1(where=(char_number=6666) rename=(char_value=char_value6666 char_points=char_points6666))
    sorted_table_1(where=(char_number=7890) rename=(char_value=char_value7890 char_points=char_points7890))
    */

    data want;
      merge &indata;
      by Customer_ID ACCOUNT_ID ;
      drop char_number;
    run;


