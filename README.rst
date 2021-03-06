================
baseball_scraper
================

`baseball_scraper` is a Python package for baseball data analysis. This package scrapes baseball-reference.com and baseballsavant.com so you don't have to. So far, the package performs four main tasks: retrieving statcast data, pitching stats, batting stats, and division standings/team records. Data is available at the individual pitch level, as well as aggregated at the season level and over custom time periods. 

Status
------

.. image:: https://travis-ci.com/spilchen/baseball_scraper.svg?branch=master
    :target: https://travis-ci.com/spilchen/baseball_scraper

Statcast
--------

*data include pitch-level features such as Perceived Velocity (PV), Spin Rate (SR), Exit Velocity (EV), pitch X, Y, and Z coordinates, and more. The function `statcast(start_dt, end_dt)` pulls this data from baseballsavant.com.*

Pull advanced metrics from Major League Baseball's Statcast system.  Statcast data include pitch-level features such as Perceived Velocity (PV), Spin Rate (SR), Exit Velocity (EV), pitch X, Y, and Z coordinates, and more. The function `statcast(start_dt, end_dt)` pulls this data from baseballsavant.com. 

::

  >>> from baseball_scraper import statcast
  >>> data = statcast(start_dt='2017-06-24', end_dt='2017-06-27')
  >>> data.head(2)
  
     index pitch_type  game_date  release_speed  release_pos_x  release_pos_z  
  0    314         CU 2017-06-27           79.7        -1.3441         5.4075
  1    332         FF 2017-06-27           98.1        -1.3547         5.4196
  
    player_name    batter   pitcher     events     ...      release_pos_y  
  0   Matt Bush  608070.0  456713.0  field_out     ...            54.8585
  1   Matt Bush  429665.0  456713.0  field_out     ...            54.3470
  
     estimated_ba_using_speedangle  estimated_woba_using_speedangle  woba_value  
  0                          0.100                            0.137         0.0
  1                          0.269                            0.258         0.0
  
     woba_denom babip_value iso_value launch_speed_angle at_bat_number pitch_number  
  0         1.0         0.0       0.0                3.0          64.0          1.0
  1         1.0         0.0       0.0                3.0          63.0          3.0  
  [2 rows x 79 columns]


If `start_dt` and `end_dt` are supplied, it will return all statcast data between those two dates. If not, it will return yesterday's data. The argument `team` may also be supplied with a team's city abbreviation (i.e. BOS) to obtain only observations for games containing that team. The optional argument `verbose` will control whether the library updates you on its progress while it pulls the data.

For a player-specific statcast query, pull pitching or batting data using the `statcast_pitcher` and `statcast_batter` functions. These take the same `start_dt` and `end_dt` arguments as the statcast function, as well as a `player_id` argument. This ID comes from MLB Advanced Media, and can be obtained using the function `playerid_lookup`. A complete example: 

::

  >>> # Find Clayton Kershaw's player id
  >>> from baseball_scraper import playerid_lookup
  >>> from baseball_scraper import statcast_pitcher
  >>> playerid_lookup('kershaw', 'clayton')
  Gathering player lookup table. This may take a moment.
  
    name_last name_first  key_mlbam key_retro  key_bbref  key_fangraphs  
  0   kershaw    clayton     477132  kersc001  kershcl01           2036
  
     mlb_played_first  mlb_played_last
  0            2008.0           2017.0
  
  >>> # His MLBAM ID is 477132, so we feed that as the player_id argument to the following function 
  >>> kershaw_stats = statcast_pitcher('2017-06-01', '2017-07-01', 477132)
  >>> kershaw_stats.head(2)
    pitch_type   game_date release_speed release_pos_x release_pos_z  
  0         SL  2017-06-29          87.2        1.0865        6.4034
  1         SL  2017-06-29          86.9        1.0195        6.4324
  
         player_name  batter  pitcher     events              description  
  0  Clayton Kershaw  458913   477132  strikeout  swinging_strike_blocked
  1  Clayton Kershaw  458913   477132       null                     ball
  
        ...       release_pos_y  estimated_ba_using_speedangle  
  0     ...             54.5463                            0.0
  1     ...             54.7625                            0.0
  
     estimated_woba_using_speedangle  woba_value woba_denom babip_value  
  0                              0.0        0.00          1           0
  1                              0.0        null       null        null
  
    iso_value launch_speed_angle at_bat_number pitch_number
  0         0               null            57            6
  1      null               null            57            5
  
  [2 rows x 78 columns]


Pitching Stats
--------------

*pitching stats for players across multiple seasons, single seasons, or during a specified time period*

This library contains two main functions for obtaining pitching data. For league-wide season-level pitching data, use the function `pitching_stats(start_season, end_season)`. This will return one row per player per season, and provide all metrics made available by FanGraphs. 

The second is `pitching_stats_range(start_dt, end_dt)`. This allows you to obtain pitching data over a specific time interval, allowing you to get more granular than the FanGraphs function (for example, to see which pitcher had the strongest month of May). This query pulls data from Baseball Reference. Note that all dates should be in `YYYY-MM-DD` format.

If you prefer Baseball Reference to FanGraphs, there is actually a third option called `pitching_stats_bref(season)`. This works the same as `pitching_stats`, but retrieves its data from Baseball Reference instead. This is typically not recommended, however, because the Baseball Reference query currently can only retrieve one season's worth of data per request.

::

  >>> from baseball_scraper import pitching_stats
  >>> data = pitching_stats(2012, 2016)
  >>> data.head()
       Season             Name     Team   Age     W    L   ERA  WAR     G    GS  
  336  2015.0  Clayton Kershaw  Dodgers  27.0  16.0  7.0  2.13  8.6  33.0  33.0
  236  2014.0  Clayton Kershaw  Dodgers  26.0  21.0  3.0  1.77  7.6  27.0  27.0
  472  2014.0     Corey Kluber  Indians  28.0  18.0  9.0  2.44  7.4  34.0  34.0
  235  2015.0     Jake Arrieta     Cubs  29.0  22.0  6.0  1.77  7.3  33.0  33.0
  256  2013.0  Clayton Kershaw  Dodgers  25.0  16.0  9.0  1.83  7.1  33.0  33.0
  
         ...      wSL/C (pi)  wXX/C (pi)  O-Swing% (pi)  Z-Swing% (pi)  
  336    ...            1.76       22.85          0.364          0.665
  236    ...            2.62         NaN          0.371          0.670
  472    ...            3.92         NaN          0.336          0.598
  235    ...            2.42         NaN          0.329          0.618
  256    ...            0.74         NaN          0.339          0.635
  
       Swing% (pi)  O-Contact% (pi)  Z-Contact% (pi)  Contact% (pi)  Zone% (pi)  
  336        0.511            0.478            0.811          0.689       0.487
  236        0.525            0.536            0.831          0.730       0.515
  472        0.468            0.485            0.886          0.744       0.505
  235        0.468            0.595            0.856          0.762       0.483
  256        0.484            0.563            0.873          0.763       0.492
  
       Pace (pi)
  336       23.4
  236       23.7
  472       24.6
  235       23.3
  256       23.4
  
  [5 rows x 299 columns]



Batting Stats
-------------

*hitting stats for players within seasons or during a specified time period*

Batting stats are obtained similar to pitching stats. The function call for getting a season-level stats is `batting_stats(start_season, end_season)`, and for a particular time range it is `batting_stats_range(start_dt, end_dt)`. The Baseball Reference equivalent for season-level data is `batting_stats_bref(season)`. 

::

  >>> from baseball_scraper import batting_stats_range
  >>> data = batting_stats_range('2017-05-01', '2017-05-08')
  >>> data.head()
            Name  Age  #days     Lev          Tm  G  PA  AB  R  H  ...    HBP  
  1   Jose Abreu   30     69  MLB-AL     Chicago  7  31  30  5  9  ...      0
  2   Lane Adams   27     69  MLB-NL     Atlanta  6   6   6  0  2  ...      0
  3   Matt Adams   28     68  MLB-NL   St. Louis  6   9   9  2  4  ...      0
  4   Jim Adduci   32     69  MLB-AL     Detroit  6  24  21  3  5  ...      0
  5  Tim Adleman   29     72  MLB-NL  Cincinnati  1   2   2  0  0  ...      0
  
     SH  SF  GDP  SB  CS     BA    OBP    SLG    OPS  mlb_ID
  1   0   0    1   0   0  0.300  0.323  0.667  0.989  547989
  2   0   0    1   1   0  0.333  0.333  0.333  0.667  572669
  3   0   0    0   0   0  0.444  0.444  0.778  1.222  571431
  4   0   0    0   0   0  0.238  0.333  0.381  0.714  451192
  5   0   0    0   0   0  0.000  0.000  0.000  0.000  534947
  
  [5 rows x 28 columns]


Fangraphs
---------

Various baseball projections are available at fangraphs.com.  You can scrape that site using the fangraphs API.  You supply it the fangraph player ID to lookup and the projection system.  It will return a DataFrame with the projections.

Note, due to the use of JavaScript on that site, we use Chrome through selenium to scrape the data.  Chrome must be installed on your system in order to use these APIs.

::

  >>> from baseball_scraper import fangraphs
  >>> from baseball_id import Lookup
  >>> player_id = Lookup.from_names(['Khris Davis']).iloc[0].fg_id
  >>> fangraphs.Scraper.instances()
  ['Steamer (RoS)', 'Steamer (Update)', 'ZiPS (Update)', 'Steamer600 (Update)', 'Depth Charts (RoS)', 'THE BAT (RoS)']
  >>> fg = fangraphs.Scraper("Steamer (RoS)")
  >>> df = fg.scrape(player_id, scrape_as=fangraphs.ScrapeType.HITTER)
  >>> df.columns
  Index(['index', 'Name', 'Team', 'G', 'PA', 'AB', 'H', '2B', '3B', 'HR', 'R',
         'RBI', 'BB', 'SO', 'HBP', 'SB', 'CS', '-1', 'AVG', 'OBP', 'SLG', 'OPS',
         'wOBA', '-1.1', 'wRC+', 'BsR', 'Fld', '-1.2', 'Off', 'Def', 'WAR',
         'playerid'],
       dtype='object')
  >>> df
  index         Name       Team   G   PA   AB   H  ...  BsR  Fld  -1.2  Off  Def  WAR  playerid
     60  Khris Davis  Athletics  56  242  214  53  ... -0.7 -0.1   NaN  4.8 -5.9  0.7      9112

  [1 rows x 32 columns]
  >>> player_id = Lookup.from_names(['Max Scherzer']).iloc[0].fg_id
  >>> df = fg.scrape(player_id, scrape_as=fangraphs.ScrapeType.PITCHER)
  >>> df.columns
  Index(['index', 'Name', 'Team', 'W', 'L', 'ERA', 'GS', 'G', 'SV', 'IP', 'H',
         'ER', 'HR', 'SO', 'BB', 'WHIP', 'K/9', 'BB/9', 'FIP', 'WAR', 'RA9-WAR',
         'playerid'],
       dtype='object')
  >>> df
  index          Name       Team  W  L   ERA  ...    K/9  BB/9   FIP  WAR  RA9-WAR  playerid
  0      5  Max Scherzer  Nationals  6  3  3.04  ...  12.36  2.13  2.93  2.2      2.4      3137

  [1 rows x 22 columns]


Game-by-Game Results and Schedule 
---------------------------------

The `baseball_reference` team scraper returns a team's game-by-game results for a given season or date range.  The resulting DataFrame includes game date, home and away teams, end result (W/L/Tie), score, winning/losing/saving pitchers, attendance, and division standing at that date.

You define the `team` when the scraper is created.  Then can reuse the scraper to scrape specific seasons or date ranges.  The team name provided is the abbreviation (i.e. NYY for New York Yankees, SEA for Seattle Mariners).

If the season argument is set to the current season, the query returns results for past games and the schedule for those that have not occurred yet. 

::

  >>> # Example: Let's take a look at the individual-game results of the 1927 Yankees
  >>> from baseball_scraper import baseball_reference
  >>> s = baseball_reference.TeamScraper()
  >>> s.set_season(1927)
  >>> data = s.scrape('NYY')
  >>> data.head()
                  Date   Tm Home_Away  Opp W/L     R   RA   Inn  W-L  Rank  \
  1    Tuesday, Apr 12  NYY      Home  PHA   W   8.0  3.0   9.0  1-0   1.0
  2  Wednesday, Apr 13  NYY      Home  PHA   W  10.0  4.0   9.0  2-0   1.0
  3   Thursday, Apr 14  NYY      Home  PHA   T   9.0  9.0  10.0  2-0   1.0
  4     Friday, Apr 15  NYY      Home  PHA   W   6.0  3.0   9.0  3-0   1.0
  5   Saturday, Apr 16  NYY      Home  BOS   W   5.0  2.0   9.0  4-0   1.0
  
         GB      Win     Loss  Save  Time D/N  Attendance  Streak
  1    Tied     Hoyt    Grove  None  2:05   D     72000.0       1
  2  up 0.5  Ruether     Gray  None  2:15   D      8000.0       2
  3    Tied     None     None  None  2:50   D      9000.0       2
  4    Tied  Pennock    Ehmke  None  2:27   D     16000.0       3
  5  up 1.0  Shocker  Ruffing  None  2:05   D     25000.0       4

  >>> # Let get the games a team plays in a given week.
  >>> import datetime as dt
  >>> s.set_date_range(dt.datetime(2019,6,2), dt.datetime(2019,6,8))
  >>> df = s.scrape('TOR')
  >>> df.head()

           Date   Tm Home_Away  Opp W/L     R  ...     Save  Time D/N  Attendance Streak Orig. Scheduled
  59 2019-06-02  TOR         @  COL   L   1.0  ...     None  2:43   D     37861.0   -6.0            None
  60 2019-06-04  TOR      Home  NYY   W   4.0  ...    Giles  3:00   N     20671.0    1.0            None
  61 2019-06-05  TOR      Home  NYY   W  11.0  ...     None  3:22   N     16609.0    2.0            None
  62 2019-06-06  TOR      Home  NYY   L   2.0  ...  Chapman  3:07   N     25657.0   -1.0            None
  63 2019-06-07  TOR      Home  ARI   L   2.0  ...     None  2:50   N     16555.0   -2.0            None

  [5 rows x 19 columns]


Team List
---------

Using the `TeamListScraper` you can pull a list of active teams and a few attributes about each from baseball-reference.  This has benefit mainly to get a list of abbreviations that baseball-reference uses for each team, as there doesn't seem to be a standard.  This comes in handy when wanting to use another baseball-reference scraper such as `TeamList` and need to input the team abbreviation.

::
    >>> from baseball_scraper import baseball_reference
    >>> tss = baseball_reference.TeamSummaryScraper()
    >>> df = tss.scrape(2019)
    >>> df.columns
    Index([u'Franchise',        u'Tm',      u'#Bat',    u'BatAge',       u'R/G',
                   u'G',        u'PA',        u'AB',         u'R',         u'H',
                  u'2B',        u'3B',        u'HR',       u'RBI',        u'SB',
                  u'CS',        u'BB',        u'SO',        u'BA',       u'OBP',
                 u'SLG',       u'OPS',      u'OPS+',        u'TB',       u'GDP',
                 u'HBP',        u'SH',        u'SF',       u'IBB',       u'LOB'],
         dtype='object')
    >>> df[df.Franchise.str.endswith("Rays")].abbrev.iloc(0)[0]
    'TBR'

Standings
---------

The `standings(season)` function gives division standings for a given season. If the current season is chosen, it will give the most current set of standings. Otherwise, it will give the end-of-season standings for each division for the chosen season. 

This function returns a list of dataframes. Each dataframe is the standings for one of MLB's six divisions. 

::

  >>> from baseball_scraper import standings
  >>> data = standings(2016)[4]
  >>> print(data)
                      Tm    W   L  W-L%    GB
  1         Chicago Cubs  103  58  .640    --
  2  St. Louis Cardinals   86  76  .531  17.5
  3   Pittsburgh Pirates   78  83  .484  25.0
  4    Milwaukee Brewers   73  89  .451  30.5
  5      Cincinnati Reds   68  94  .420  35.5


ESPN
----

With the ESPN scraper you can pull probable starters for a given date range.  This can be useful to see the two-start pitchers for a given week and their team matchups.

::

  >>> from baseball_scraper import espn
  >>> from datetime import datetime
  >>> es = espn.ProbableStartersScraper(datetime(2019,8,5), datetime(2019,8,11))
  >>> df = es.scrape()
  >>> df.head(10)
          Date              Name  espn_id opponent
  0 2019-08-05   Sandy Alcantara    35241      NYM
  1 2019-08-05      Jacob deGrom    32796      MIA
  2 2019-08-05      Jordan Lyles    31061      PIT
  3 2019-08-05     Dario Agrazal    39813      MIL
  4 2019-08-05   Masahiro Tanaka    33150      BAL
  5 2019-08-05      Gabriel Ynoa    33651      NYY
  6 2019-08-05     Lucas Giolito    32697      DET
  7 2019-08-05  Spencer Turnbull    33732      CHW
  8 2019-08-05   Mike Montgomery    31092      BOS
  9 2019-08-05     Rick Porcello    29966       KC
  >>> df[df.duplicated(['espn_id'])].Name
  127     Masahiro Tanaka
  128    Jacob Waguespack
  130       Rick Porcello
  131     Vince Velasquez
  132     Jeff Samardzija
  133     Mike Montgomery
  134    Spencer Turnbull
  135         Mike Soroka
  136     Sandy Alcantara
  139       Jake Odorizzi
  146      Kyle Hendricks
  153      Charlie Morton
  155      Andrew Cashner
  157      Trent Thornton
  158         Jakob Junis
  159       Daniel Norris
  160        Jacob deGrom
  161           Max Fried
  162     Jordan Yamamoto
  163          Jon Lester
  164       Luis Castillo
  165       Chris Bassitt
  166       Lucas Giolito
  167          Mike Minor
  168        Jordan Lyles
  169         Zach Plesac
  170        Jose Berrios
  171       Dario Agrazal
  172       Michael Wacha
  173      German Marquez
  174      Dinelson Lamet
  175       Merrill Kelly
  176        Wade LeBlanc
  177        Jake Arrieta
  Name: Name, dtype: object
  >>> df[df.Name == "Charlie Morton"]
            Date            Name  espn_id opponent
  15  2019-08-05  Charlie Morton    29155      TOR
  153 2019-08-10  Charlie Morton    29155      SEA


Complete Documentation
----------------------

So far this has provided a basic overview of what this package can do and how you can use it. For full documentation on available functions and their arguments, see the [docs](https://github.com/spilchen/baseball_scraper/tree/master/docs) folder. 

Installation
------------

To install baseball_scraper, simply run 

::

  pip install baseball_scraper

or, for the version currently on the repo (which may at times be more up to date):

::

  git clone https://github.com/spilchen/baseball_scraper
  cd baseball_scraper
  python setup.py install


Testing
-------

We use pytest for testing the package.  It can be invoked as follows:

::
  python setup.py test


Dependencies
------------

This library depends on: Pandas, NumPy, bs4 (beautiful soup), and Requests. 
