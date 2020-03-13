# utl-Rolling-four-month-median-by-group
SAS Forum: Rolling four month median by group
    Rolling four month median by group

    github related
    https://github.com/rogerjdeangelis/utl-moving-ten-month-average-by-group
    https://github.com/rogerjdeangelis/utl-Rolling-four-month-median-by-group

    SAS Forum
    https://tinyurl.com/rkrmzok
    https://communities.sas.com/t5/SAS-Programming/Calculating-a-rolling-median-with-varying-numbers-of/m-p/631976

    For other examples of 'proc expand', hand coding and R solutions
    https://github.com/rogerjdeangelis?tab=repositories&q=rolling&type=&language=
    https://github.com/rogerjdeangelis?tab=repositories&q=+moving+&type=&language=

    Related to
    SAS-L: https://listserv.uga.edu/cgi-bin/wa?A2=SAS-L;a4df8616.2003a

    Rolling moving 4 month average by group

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    filename bin "d:/bin/binmat.bin" lrecl=32000 recfm=f;
    data have;
      retain id month;
      format month yymm7.;
      call streaminit(54321);
      do id=101 to 102;
         do mnth=1 to 15;
             sumResponse=int(10*rand('uniform'));
             month=intnx('month', '01JAN2010'd, mnth-1);
             file bin;
             put (id sumresponse) (2*rb8.) @@ ;
             if id=102 and mnth>13 then leave;
             output;
          end;
      end;
    run;quit;


    HAVE total obs=28

    Obs     ID     MONTH     MNTH    SUMRESPONSE

      1    101    2010M01      1          4
      2    101    2010M02      2          5
      3    101    2010M03      3          7
      4    101    2010M04      4          1
      5    101    2010M05      5          3
      6    101    2010M06      6          6
      7    101    2010M07      7          8
      8    101    2010M08      8          0
      9    101    2010M09      9          9
     10    101    2010M10     10          0
     11    101    2010M11     11          1
     12    101    2010M12     12          2
     13    101    2011M01     13          1
     14    101    2011M02     14          5
     15    101    2011M03     15          4
     16    102    2010M01      1          6
     17    102    2010M02      2          8
     18    102    2010M03      3          8
     19    102    2010M04      4          3
     20    102    2010M05      5          7
     21    102    2010M06      6          4
     22    102    2010M07      7          3
     23    102    2010M08      8          0
     24    102    2010M09      9          1
     25    102    2010M10     10          8
     26    102    2010M11     11          8
     27    102    2010M12     12          3
     28    102    2011M01     13          8


    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    WINDOW IS 4 MONTHS


    Up to 40 obs WORK.BIN total obs=28

     Obs     ID    MONTH    MNTH    SUMRESPONSE    AVE4

       1    101    18263      1          4           .
       2    101    18294      2          5           .
       3    101    18322      3          7           .
       4    101    18353      4          1          4.5  1 4 5 7 Median is (4+5)/2 = 4.5
       5    101    18383      5          3          4.0  1 3 5 7 Median is (3+5)/2 = 4
       6    101    18414      6          6          4.5
       7    101    18444      7          8          4.5
       8    101    18475      8          0          4.5
       9    101    18506      9          9          7.0
      10    101    18536     10          0          4.0
      11    101    18567     11          1          0.5
      12    101    18597     12          2          1.5
      13    101    18628     13          1          1.0
      14    101    18659     14          5          1.5
      15    101    18687     15          4          3.0
      16    102    18263      1          6           .
      17    102    18294      2          8           .
      18    102    18322      3          8           .
      19    102    18353      4          3          7.0  3 6 8 8 Median is 7
      20    102    18383      5          7          7.5
      21    102    18414      6          4          5.5
      22    102    18444      7          3          3.5
      23    102    18475      8          0          3.5
      24    102    18506      9          1          2.0
      25    102    18536     10          8          2.0
      26    102    18567     11          8          4.5
      27    102    18597     12          3          5.5
      28    102    18628     13          8          8.0

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    * make data same as above code;
    filename bin "d:/bin/binmat.bin" lrecl=32000 recfm=f;
    data have;
      retain id month;
      format month yymm7.;
      call streaminit(54321);
      do id=101 to 102;
         do mnth=1 to 15;
             sumResponse=int(10*rand('uniform'));
             month=intnx('month', '01JAN2010'd, mnth-1);
             file bin;
             put (id sumresponse) (2*rb8.) @@ ;
             if id=102 and mnth>13 then leave;
             output;
          end;
      end;
    run;quit;

    * widow size;
    %let window=4;

    proc sql;
      select count(*)*2 into :_obs from have
    ;quit;


    %utl_submit_r64(resolve('
    library(SASxport);
    library(TTR);
    library(dplyr);
    library(data.table);
    read.from <- file("d:/bin/binmat.bin", "rb");
    mat <- readBin(read.from, n=&_obs., "double");
    str(mat);
    close(read.from);
    mat <- as.data.table(matrix(mat,&_obs./2,2,byrow=T));
    mat[1:28];
    ra<-mat %>% group_by(V1) %>% mutate(ra = runMedian(V2, &window));
    want<-as.vector(ra$ra);
    outbin <- "d:/bin/want.bin";
    writeBin(want, file(outbin, "wb"), size=8);
    '));


    filename bin "d:/bin/want.bin" lrecl=8 recfm=f;
    data want ;
     infile bin;
     set hAVE;
     input ave10 rb8. @@;
     put ave10;
    run;quit;

