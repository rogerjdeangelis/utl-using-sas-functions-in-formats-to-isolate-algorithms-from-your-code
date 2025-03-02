# utl-using-sas-functions-in-formats-to-isolate-algorithms-from-your-code
Using sas functions in formats to isolate algorithms from your code 
    %let pgm=utl-using-sas-functions-in-formats-to-isolate-algorithms-from-your-code;

    %stop_submission;

    Best run in classic editor enviorment?
    You may need the sasfont?

    Using sas functions in formats to isolate algorithms from your code

      1. regular wxpressions and chaining formats
      2. convert fahrenheit temperatures censius
      3. check if varible has lengh=10
      4  mean age by state when state part of smart key
         groupformat and sql
      5. fcmp and proc sql mean age by state when state part of smart key
      6. convering pounds, pesos and dollars to amount
      7. 1dentify male adults and male children
         how to create a function with multiple arguments
      8  related repos

    Inspired by

    The sas guru of gurus Rick Langston
    https://www.lexjansen.com/pharmasug/2012/TF/PharmaSUG-2012-TF21-SAS.pdf

    https://goo.gl/L9FcJO
    http://stackoverflow.com/questions/41183167/check-for-length-of-a-character-string-in-sas-proc-format

    /***************************************************************************************************************************/
    /*                               |                                                       |                                 */
    /*  INPUT                        |                   PROCESS                             |  OUTPUT                         */
    /*  =====                        |                   =======                             |  ======                         */
    /*                               |                                                       |                                 */
    /*                               |                                                       |                                 */
    /*  HAVE                         | 1. REGULAR WXPRESSIONS AND CHAINING FORMATS           |                                 */
    /*                               |                                                       |   WANT                          */
    /*    X                          |                                                       |                                 */
    /*                               | * In case you rerun;                                  |     X                           */
    /*    XA                         | proc catalog catalog=work.formats;                    |                                 */
    /*    XB                         |   delete xa xb xc / entrytype=infmtc;                 |     X1                          */
    /*    XC                         | run;quit;                                             |     X2                          */
    /*    X4                         |                                                       |     X3                          */
    /*                               | proc format;                                          |     X4                          */
    /*  data have;                   | invalue $xa 's/XA/X1/' (regexpe)=_same_ other=[$XB.]; |                                 */
    /*     length x $2;              | invalue $xb 's/XB/X2/' (regexpe)=_same_ other=[$XC.]; |                                 */
    /*     input x$;                 | invalue $xc 's/XC/X3/' (regexpe)=_same_ ;             |                                 */
    /*  cards4;                      | run;quit;                                             |                                 */
    /*  XA                           |                                                       |                                 */
    /*  XB                           | data want;                                            |                                 */
    /*  XC                           |  set have;                                            |                                 */
    /*  X4                           |  x=input(x,$xa.);                                     |                                 */
    /*  ;;;;                         | run;quit;                                             |                                 */
    /*  run;quit;                    |                                                       |                                 */
    /*                               |                                                       |                                 */
    /*-------------------------------------------------------------------------------------------------------------------------*/
    /*                               |                                                       |                                 */
    /*                               |                                                       |                                 */
    /*                               |                                                       |                                 */
    /*                               |                                                       |                                 */
    /*                               |                                                       |                 RESULT          */
    /*   PHASE      DEG_FX           |  2. CONVERT FAHRENHEIT TEMPERATURES CENSIUS           |  PHASE   DEG_C   DEG_F          */
    /*                               |  ===========================================          |                                 */
    /*  freezing       32            |                                                       | freezing 0°C     32°F           */
    /*  sametemp      -40            |  proc fcmp outlib=work.functions.demo;                | sametemp -40°C   -40°F          */
    /*  boiling       212            |    deletefunc ctof;                                   | boiling  100°C   212°F          */
    /*                               |    deletefunc ftoc;                                   |                                 */
    /*  data have;                   |  run;quit;                                            |                                 */
    /*   input phase$ deg_fx;        |                                                       |                                 */
    /*  cards4;                      |  options cmplib=work.functions;                       |                                 */
    /*  freezing 32                  |                                                       |                                 */
    /*  sametemp -40                 |  proc datasets lib=work nodetails nolist;             |                                 */
    /*  boiling 212                  |   delete want;                                        |                                 */
    /*  ;;;;                         |  run;quit;                                            |                                 */
    /*  run;quit;                    |                                                       |                                 */
    /*                               |  proc catalog cat=work.formats;                       |                                 */
    /*                               |    delete ftoc / entrytype=format;                    |                                 */
    /*                               |  ;quit;run;                                           |                                 */
    /*                               |                                                       |                                 */
    /*                               |  proc fcmp outlib=work.functions.demo;                |                                 */
    /*                               |  function ctof(c) $;                                  |                                 */
    /*                               |    return(cats(((9*c)/5)+32,'°F'));                   |                                 */
    /*                               |  endsub;                                              |                                 */
    /*                               |  function ftoc(f) $;                                  |                                 */
    /*                               |    return(cats((f-32)*5/9,'°C'));                     |                                 */
    /*                               |  endsub;                                              |                                 */
    /*                               |  run;                                                 |                                 */
    /*                               |                                                       |                                 */
    /*                               |  proc format;                                         |                                 */
    /*                               |  value ctof (default=10) other=[ctof()];              |                                 */
    /*                               |  value ftoc (default=10) other=[ftoc()];              |                                 */
    /*                               |  run;                                                 |                                 */
    /*                               |                                                       |                                 */
    /*                               |  data want;                                           |                                 */
    /*                               |    set have;                                          |                                 */
    /*                               |    deg_c=put(deg_fx,ftoc.);                           |                                 */
    /*                               |    deg_f=cats(put(deg_fx,5.),'°F');                   |                                 */
    /*                               |    drop deg_fx;                                       |                                 */
    /*                               |  run;quit;                                            |                                 */
    /*                               |                                                       |                                 */
    /*-------------------------------------------------------------------------------------------------------------------------*/
    /*                               |                                                       |                                 */
    /*   CARDNO                      |  3. CHECK IF VARIABLE HAS LENGH=10                    |  WANT                           */
    /*                               |  ================================                     |                                 */
    /*   1                           |                                                       |   CARDNO       LENGT10          */
    /*   12                          |  options cmplib=work.functions;                       |                                 */
    /*   123                         |                                                       |   1             ERROR           */
    /*   12345                       |  proc fcmp outlib=work.functions.demo;                |   12            ERROR           */
    /*   123456                      |    deletefunc tenbytes;                               |   123           ERROR           */
    /*   1234567                     |  run;quit;                                            |   12345         ERROR           */
    /*   123456789                   |                                                       |   123456        ERROR           */
    /*   1234567890                  |  proc datasets lib=work nodetails nolist;             |   1234567       ERROR           */
    /*   12345678901                 |   delete want;                                        |   123456789     ERROR           */
    /*   123456789012                |  run;quit;                                            |   1234567890    LENGTH=10       */
    /*                               |                                                       |   12345678901   ERROR           */
    /*                               |  proc fcmp outlib=work.functions.demo;                |   123456789012  ERROR           */
    /*  data have;                   |   function tenbytes(charval $) $;                     |                                 */
    /*  length cardno $16;           |    length retval $16;                                 |                                 */
    /*  input cardno;                |    if length(charval) = 10 then retval='LENGTH=10';   |                                 */
    /*  cards4;                      |    else retval='ERROR';                               |                                 */
    /*  1                            |    return(retval);                                    |                                 */
    /*  12                           |   endsub;                                             |                                 */
    /*  123                          |  run;quit;                                            |                                 */
    /*  12345                        |                                                       |                                 */
    /*  123456                       |  proc format;                                         |                                 */
    /*  1234567                      |    value $chk10f                                      |                                 */
    /*  123456789                    |    low-high = [tenbytes()];                           |                                 */
    /*  1234567890                   |  quit;                                                |                                 */
    /*  12345678901                  |                                                       |                                 */
    /*  123456789012                 |  data want;                                           |                                 */
    /*  ;;;;                         |    set have;                                          |                                 */
    /*  run;quit;                    |    FormattedStatus=put(cardno,$chk10f.);              |                                 */
    /*                               |    format cardno $16. cardno;                         |                                 */
    /*                               |  run;                                                 |                                 */
    /*                               |                                                       |                                 */
    /*                               |  proc print data=want;                                |                                 */
    /*                               |  run;quit;                                            |                                 */
    /*                               |                                                       |                                 */
    /*-------------------------------------------------------------------------------------------------------------------------*/
    /*                               |                                                       |                                 */
    /*                               |                                                       |                                 */
    /*                |  OUTPUT      |  4  MEAN AGE BY STATE WHEN STATE PART OF SMART KEY    |    WANT                         */
    /* SEX   ID   AGE |  MEAN AGE    |  =================================================    |                                 */
    /*                |              |                                                       |      ID      GRPAVG             */
    /*  M  1_AL.3  50 | ALABAMA      |  * IN CASE YOU RERUN;                                 |                                 */
    /*  F  4.AL.3  40 | (50+40)/2=45 |                                                       |    4.AL.3      45               */
    /*                |              |  options cmplib=work.functions;                       |    D_VT_D      50               */
    /*  M  A-VT-A  60 | VERMONT      |                                                       |    3-NY-3      50               */
    /*  F  D_VT_D  40 | (60+40)/2=50 |  proc fcmp outlib=work.functions.demo;                |    B.TX.B      50               */
    /*                |              |    deletefunc keycut;                                 |                                 */
    /*  M  2.NY.3  50 |              |  run;quit;                                            |                                 */
    /*  F  3-NY-3  50 |              |                                                       |                                 */
    /*                |              |  proc datasets lib=work nodetails nolist;             |                                 */
    /*  M  C_TX_C  45 |              |   delete want;                                        |                                 */
    /*  F  B.TX.B  55 |              |  run;quit;                                            |                                 */
    /*                               |                                                       |                                 */
    /*                               |  proc catalog cat=work.formats;                       |                                 */
    /*  data have;                   |    delete cutkey / entrytype=format;                  |                                 */
    /*    input sex$ ID$ age;        |  ;quit;run;                                           |                                 */
    /*  cards4;                      |                                                       |                                 */
    /*  M 1_AL.3 50                  |  * THIS IS THE WORKING CODE OPERATIVE CODE;           |                                 */
    /*  F 4.AL.3 40                  |                                                       |                                 */
    /*  M A-VT-A 60                  |  proc fcmp outlib=work.functions.demo;                |                                 */
    /*  F D_VT_D 40                  |    function keycut(key $) $8;                         |                                 */
    /*  M 2.NY.3 50                  |      cutkey=substr(key,3,2);                          |                                 */
    /*  F 3-NY-3 50                  |    return(cutkey);                                    |                                 */
    /*  M C_TX_C 45                  |  endsub;                                              |                                 */
    /*  F B.TX.B 55                  |  run;quit;                                            |                                 */
    /*  ;;;;                         |                                                       |                                 */
    /*  run;quit;                    |  proc format;                                         |                                 */
    /*                               |    value $cutkey other=[keycut()];                    |                                 */
    /*                               |  run;quit;                                            |                                 */
    /*                               |                                                       |                                 */
    /*                               |  data want ;                                          |                                 */
    /*                               |     retain grptot 0;                                  |                                 */
    /*                               |     format id $cutkey.;                               |                                 */
    /*                               |     set have;                                         |                                 */
    /*                               |     *put id= $cutkey.;                                |                                 */
    /*                               |     by id notsorted GROUPFORMAT;                      |                                 */
    /*                               |     *put age=;                                        |                                 */
    /*                               |     grptot+age;                                       |                                 */
    /*                               |     *put grptot=;                                     |                                 */
    /*                               |     if last.id then do;                               |                                 */
    /*                               |       grpavg=grptot/2;                                |                                 */
    /*                               |       output;                                         |                                 */
    /*                               |       grptot=0;                                       |                                 */
    /*                               |     end;                                              |                                 */
    /*                               |     keep id grpavg;                                   |                                 */
    /*                               |  run;quit;                                            |                                 */
    /*-------------------------------------------------------------------------------------------------------------------------*/
    /*                               |                                                       |                                 */
    /*                               |                                                       |                                 */
    /*                               |  5  FCMP AND PROC SQL                                 |    WANT                         */
    /*                               |  ====================                                 |                                 */
    /*                               |                                                       |    STATE    AVGAGE              */
    /*                               |  proc sql;                                            |                                 */
    /*                               |    create                                             |     AL        45                */
    /*                               |      table want as                                    |     NY        50                */
    /*                               |    select                                             |     TX        50                */
    /*                               |      state                                            |     VT        50                */
    /*                               |     ,avg(age) as avgage                               |                                 */
    /*                               |    from                                               |                                 */
    /*                               |      (                                                |                                 */
    /*                               |       select                                          |                                 */
    /*                               |         keycut(id) as state                           |                                 */
    /*                               |        ,age                                           |                                 */
    /*                               |       from                                            |                                 */
    /*                               |         have                                          |                                 */
    /*                               |      )                                                |                                 */
    /*                               |      group                                            |                                 */
    /*                               |         by state                                      |                                 */
    /*                               |  ;quit;                                               |                                 */
    /*                               |                                                       |                                 */
    /*-------------------------------------------------------------------------------------------------------------------------*/
    /*                               |                                                       |                        RESULT   */
    /*   SALARY       CURRENCY       |  6  CONVERING POUNDS, PESOS AND DOLLARS TO AMOUNT     |  SALARY      CURRENCY  AMOUNT   */
    /*                               |  ================================================     |                                 */
    /* $1,473.33       dollars       |                                                       |                                 */
    /* £1,560.00       pounds        |  options cmplib=work.functions;                       | $1,473.33     dollars  1473.33  */
    /* 1560,55 MX$     Pesos         |                                                       | £1,560.00     pounds   1560.00  */
    /* 1560,55 Mex$    Pesos         |  proc fcmp outlib=work.functions.demo;                | 1560,55 MX$   Pesos    1560.55  */
    /*                               |    deletefunc currency;                               | 1560,55 Mex$  Pesos    1560.55  */
    /*                               |  run;quit;                                            |                                 */
    /* data have ;                   |                                                       |                                 */
    /* informat salary currency $12.;|  proc catalog cat=work.formats;                       |                                 */
    /*  input salary & currency ;    |    delete currency / entrytype=infmt;                 |                                 */
    /* cards4;;                      |  ;quit;run;                                           |                                 */
    /* $1,473.33  dollars            |                                                       |                                 */
    /* £1,560.00  pounds             |                                                       |                                 */
    /* 1560,55 MX$  Pesos            |  proc fcmp outlib=work.functions.currency;            |                                 */
    /* 1560,55 Mex$  Pesos           |  function currency(txt $) $16;                        |                                 */
    /* ;;;;                          |    select;                                            |                                 */
    /* run;quit;                     |      when (substr(txt,1,1)='$')                       |                                 */
    /*                               |          amt=input(substr(txt,2),comma12.2);          |                                 */
    /*                               |      when (substr(txt,1,1)='£')                       |                                 */
    /*                               |          amt=input(substr(txt,2),comma12.2);          |                                 */
    /*                               |      when (index(txt,'M'))                            |                                 */
    /*                               |          amt=input(scan(txt,1,' '),numx12.2);         |                                 */
    /*                               |      otherwise amt=.;                                 |                                 */
    /*                               |    end;                                               |                                 */
    /*                               |    return(amt);                                       |                                 */
    /*                               |  endsub;                                              |                                 */
    /*                               |  ;run;quit;                                           |                                 */
    /*                               |                                                       |                                 */
    /*                               |  proc format;                                         |                                 */
    /*                               |  invalue currency (default=20)                        |                                 */
    /*                               |  other=[currency()];                                  |                                 */
    /*                               |  run;                                                 |                                 */
    /*                               |                                                       |                                 */
    /*                               |  data want ;                                          |                                 */
    /*                               |    set have ;                                         |                                 */
    /*                               |    amount=input(salary,currency.) ;                   |                                 */
    /*                               |  run;quit;                                            |                                 */
    /*                               |                                                       |                                 */
    /*-------------------------------------------------------------------------------------------------------------------------*/
    /*                               |                                                       |                                 */
    /*                               |                                                       |                                 */
    /*   NAME   SEX AGE              |  7. 1DENTIFY MALE AND FEMALE ADULTS                   |  NAME  SEX AGE    STATUS        */
    /*                               |  ==================================                   |                                 */
    /*  Joyce    F   11              |                                                       | Joyce   F   11 female child     */
    /*  Thomas   M   11              |  * JUST IN CASE YOU RERUN;                            | Thomas  M   11 male child       */
    /*  Barbara  F   13              |                                                       | Barbara F   13 female child     */
    /*  Philip   M   76              |  proc datasets lib=work nodetails nolist;             | Philip  M   76 male child       */
    /*                               |   delete want;                                        |                                 */
    /*  data have;                   |  run;quit;                                            |                                 */
    /*    input name$                |                                                       |                                 */
    /*       sex$ age;               |  proc fcmp outlib=work.functions.demo;                |                                 */
    /*  cards4;                      |    deletefunc apnage;                                 |                                 */
    /*  Joyce F 11                   |    deletefunc agesex;                                 |                                 */
    /*  Thomas M 11                  |  run;quit;                                            |                                 */
    /*  Barbara F 13                 |                                                       |                                 */
    /*  Philip M 76                  |  proc catalog cat=work.formats;                       |                                 */
    /*  ;;;;                         |    delete agesex / entrytype=formatc;                 |                                 */
    /*  run;quit;                    |  ;quit;run;                                           |                                 */
    /*                               |                                                       |                                 */
    /*                               |  *  ONE TIME FORMAT AND FUNCTIONS;                    |                                 */
    /*                               |                                                       |                                 */
    /*                               |  options cmplib = work.functions;                     |                                 */
    /*                               |                                                       |                                 */
    /*                               |  proc fcmp outlib=work.functions.demo;                |                                 */
    /*                               |                                                       |                                 */
    /*                               |    function apnage(sex $, age ) $16;                  |                                 */
    /*                               |         * determines the length return argument;      |                                 */
    /*                               |         apn=cats(sex,put(age,z11.));                  |                                 */
    /*                               |       return(apn);                                    |                                 */
    /*                               |                                                       |                                 */
    /*                               |     endsub;                                           |                                 */
    /*                               |                                                       |                                 */
    /*                               |     function agesex(apn $ ) $16;                      |                                 */
    /*                               |                                                       |                                 */
    /*                               |       sex=substr(apn,1,1);                            |                                 */
    /*                               |       age=input(substr(apn,2),2.);                    |                                 */
    /*                               |                                                       |                                 */
    /*                               |       if (sex eq 'M') then do;                        |                                 */
    /*                               |          if (age le 18) then status='male child  ';   |                                 */
    /*                               |          else status = 'male adult';                  |                                 */
    /*                               |        end;                                           |                                 */
    /*                               |        else do;                                       |                                 */
    /*                               |          if (age le 18) then status='female child';   |                                 */
    /*                               |          else status = 'female adult';                |                                 */
    /*                               |        end;                                           |                                 */
    /*                               |       return(status);                                 |                                 */
    /*                               |                                                       |                                 */
    /*                               |      endsub;                                          |                                 */
    /*                               |                                                       |                                 */
    /*                               |     ;run;quit;                                        |                                 */
    /*                               |                                                       |                                 */
    /*                               |     proc format;                                      |                                 */
    /*                               |     value $agesex (default=16) other=[agesex()];      |                                 */
    /*                               |     run;quit;                                         |                                 */
    /*                               |                                                       |                                 */
    /*                               |     * OPERATIVE CODE;                                 |                                 */
    /*                               |                                                       |                                 */
    /*                               |     data want;                                        |                                 */
    /*                               |      set have;                                        |                                 */
    /*                               |      status=put(apnage(sex,age),$agesex.);            |                                 */
    /*                               |     run;quit;                                         |                                 */
    /*                               |                                                       |                                 */
    /***************************************************************************************************************************/

    /*___              _       _           _
     ( _ )    _ __ ___| | __ _| |_ ___  __| |  _ __ ___ _ __   ___  ___
     / _ \   | `__/ _ \ |/ _` | __/ _ \/ _` | | `__/ _ \ `_ \ / _ \/ __|
    | (_) |  | | |  __/ | (_| | ||  __/ (_| | | | |  __/ |_) | (_) \__ \
     \___/   |_|  \___|_|\__,_|\__\___|\__,_| |_|  \___| .__/ \___/|___/
                                                       |_|
    */

    REPOS
    https://github.com/rogerjdeangelis/utl-attempt-to-call-fcmp-containing-pokelong-from-sql
    https://github.com/rogerjdeangelis/utl-calculate-regression-coeficients-in-base-sas-fcmp-proc-reg-r-and-python
    https://github.com/rogerjdeangelis/utl-create-and-invert-a-two-dimensional-numeric-array-from-a-sas-dataset-r-sas-fcmp
    https://github.com/rogerjdeangelis/utl-datastep-in-memory-matrix-and-submatrix-row-and-column-reductions-summations-fcmp
    https://github.com/rogerjdeangelis/utl-extracting-sas-meta-data-using-sas-macro-fcmp-and-dosubl
    https://github.com/rogerjdeangelis/utl-flip-a-file-upside-down-using-recent-fcmp-dynamic-array-by-Bart-Jablonski
    https://github.com/rogerjdeangelis/utl-implementation-and-benchmarks-for-heapsort-mergesort-quicksort-sortn-fcmpsort
    https://github.com/rogerjdeangelis/utl-many-interfaces-to-python-open-code-and-within-datastep-fcmp-wps
    https://github.com/rogerjdeangelis/utl-numeric-to-numeric-formats-using-fcmp-and-proc-format
    https://github.com/rogerjdeangelis/utl-proof-of-concept-using-dosubl-to-create-a-fcmp-like-function-for-a-rolling-sum-of-size-three
    https://github.com/rogerjdeangelis/utl-push-and-pop-words-off-a-string-sas-fcmp
    https://github.com/rogerjdeangelis/utl-round-up-to-the-nearest-mutiple-of-n-using-fcmp-code-fx-equals-ceil-x-divided-by-n-times-n
    https://github.com/rogerjdeangelis/utl-sas-array-macro-fcmp-or-dosubl-take-your-choice
    https://github.com/rogerjdeangelis/utl-sas-fcmp-hash-stored-programs-python-r-functions-to-find-common-words
    https://github.com/rogerjdeangelis/utl-simple-datastep-replacement-for-fcmp-use-stored-program-
    https://github.com/rogerjdeangelis/utl-very-fast-elegant-examples-of-recursion-using-fcmp-by-Bartosz-Jablonski
    https://github.com/rogerjdeangelis/utl_sort_summarize_set_merge_using_functions_in_formats_groupformat_fcmp






    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
