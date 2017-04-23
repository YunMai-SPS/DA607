Sporting and Data Analytics: An Exploration into Formula One
================
Yun Mai
April 22, 2017

Motivation
----------

I choose Formula One as the topic of the final project because I want to understand how to entertain a data driven sport and of possible to learn how this industry work. To me, the most fascinated part of this sport is its technical aspect as it represents both the advanced automobile and aeronautical engineering. While Formula One could be seen as automobile companies showcasing their ability to perform in racing sport, it brings the fast evolution of the technologies behind the sport. The racing telemetry collect data like the speed, stability, tire wear, and aerodynamics etc. combined with data analytics allows engineers to evaluate the performance of cars around the racing track and figure out what needs to change. The audience could anticipate the enhanced engineering by each passing season. Therefore data is the trade secret for each team because whether teams can shave hundredths of seconds off their lap times will rely on the details of those data. Another reason I like to investigate Formula One is that it is a game of number. For example, 1.5 GB of data will generate for each car per race for McLaren. Each weekend the Grand Prix racing results broadcast on TV so that audience can follow the race and be updated. More data including practice laps, warming up are available at Formula One Live Timing. So there are plenty of data for me to do interesting analysis in this sport.

Goal
----

The goal of this project is to apply R language and MySQL I've learned in Data Acquisition and Management Course to collect, structure and visualize data in the context of Formula One sporting. At the same time, I hope I can learn the history of this sport and learn how this industry works through digging the data of racing results. Formula One rule becomes more and more complicated in the regulation of the costs, safety. Thought this gives the dedicated fans more fun, it makes it difficult to someone who is new to this sport to enjoy it right away. To knowing how technical and sporting regulations shape this sport, I will extract the information from the archive and find out quantification analysis.

Data Science Workflow
---------------------

To obtain the goal, I will use OSEMN model. That is, I will execute a data science workflow that includes:

Obtaining data Scrubbing data Exploring data Modeling data iNterpreting data

Obtaining data
--------------

I will use the following data source:

1.Ergast Developer API (ergast.com/mrd/)

2.FIA archive from 2012 to 2016 (<http://www.fia.com/f1-archives>) and the current year data in this URL: <http://www.fia.com/events/fia-formula-one-world-championship/season-2017/2017-fia-formula-one-world-championship>.

3.Formula1.com archive from 1950-2016 (<https://www.formula1.com/en/results.html/1950/races/94/great-britain/race-result.html>) (Great Britain) and the and the current year data in this URL: <https://www.formula1.com/en/results.html/2017/races/959/australia/fastest-laps.html>

4.Some statistic at f1 database (<http://www.f1db.de/>)

For API, the XML or JSON file will be downloaded and stored. For PDF, I will use PDFTables to extract data and convert to CSV file. R packages "RCurl", "jsonlite", ""XML will be used in downloading data from the websites.

The data could be large and I will upload the data to Amazon Relational Database Service (RDS) Free Tier (<https://aws.amazon.com/rds/free/>)

    ## Loading required package: bitops

    ## 
    ## Attaching package: 'tidyr'

    ## The following object is masked from 'package:RCurl':
    ## 
    ##     complete

    ## Warning: package 'dplyr' was built under R version 3.3.3

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    ## Warning: package 'ggplot2' was built under R version 3.3.3

**The sample raw data extract from Ergast Developer API is shown below.**

| Driver                               | Constructor       | Q1       | Q2       | Q3       |
|:-------------------------------------|:------------------|:---------|:---------|:---------|
| 19FelipeMassa1981-04-25Brazilian     | FerrariItalian    | 1:25.994 | 1:26.192 | 1:27.617 |
| HeikkiKovalainen1981-10-19Finnish    | McLarenBritish    | 1:26.736 | 1:26.290 | 1:27.808 |
| 44LewisHamilton1985-01-07British     | McLarenBritish    | 1:26.192 | 1:26.477 | 1:27.923 |
| 7KimiRäikkönen1979-10-17Finnish      | FerrariItalian    | 1:26.457 | 1:26.050 | 1:27.936 |
| RobertKubica1984-12-07Polish         | BMW SauberGerman  | 1:26.761 | 1:26.129 | 1:28.390 |
| MarkWebber1976-08-27Australian       | Red BullAustrian  | 1:26.773 | 1:26.466 | 1:28.417 |
| 14FernandoAlonso1981-07-29Spanish    | RenaultFrench     | 1:26.836 | 1:26.522 | 1:28.422 |
| JarnoTrulli1974-07-13Italian         | ToyotaJapanese    | 1:26.695 | 1:26.822 | 1:28.836 |
| NickHeidfeld1977-05-10German         | BMW SauberGerman  | 1:27.107 | 1:27.607 | 1:28.882 |
| DavidCoulthard1971-03-27British      | Red BullAustrian  | 1:26.939 | 1:26.520 | 1:29.959 |
| 6NicoRosberg1985-06-27German         | WilliamsBritish   | 1:27.367 | 1:27.012 | NA       |
| RubensBarrichello1972-05-23Brazilian | HondaJapanese     | 1:27.355 | 1:27.219 | NA       |
| 22JensonButton1980-01-19British      | HondaJapanese     | 1:27.428 | 1:27.298 | NA       |
| 5SebastianVettel1987-07-03German     | Toro RossoItalian | 1:27.442 | 1:27.412 | NA       |
| TimoGlock1982-03-18German            | ToyotaJapanese    | 1:26.614 | 1:27.806 | NA       |
| KazukiNakajima1985-01-11Japanese     | WilliamsBritish   | 1:27.547 | NA       | NA       |
| NelsonPiquet Jr.1985-07-25Brazilian  | RenaultFrench     | 1:27.568 | NA       | NA       |
| SébastienBourdais1979-02-28French    | Toro RossoItalian | 1:27.621 | NA       | NA       |
| GiancarloFisichella1973-01-14Italian | Force IndiaIndian | 1:27.807 | NA       | NA       |
| 99AdrianSutil1983-01-11German        | Force IndiaIndian | 1:28.325 | NA       | NA       |

Scrubbing data
--------------

The raw data will be cleaned. Some Heading of XML or JSON file will be removed and the data will be converted to a data frame. Some number will be converted to numeric if they are presented as characters. Tidying will be performed to transform the data to a structure that is easy for statistical analysis. R packages "stringr", "tidyr", "dplyr", "knitr", "RMySQL" will be used in data cleaning, transforming and storage.

Exploring data
--------------

With the clean data in hands, I will first check the distributions of the lap times of each driver by plotting histogram or boxplot. The basic statistical analysis could be done by the summary function. Also, I could view the stint time segmentation with the time elapse for all drivers.

Modeling data and iNterpreting data
-----------------------------------

Reproduce how each driver uses the track in the practice by plotting the accumulated time derived from the lap time record.

Lap chart: position changes of each driver at each run of qualifying.

Race chart: plot the gap to the leader of each driver at each lap to view the relative position change with the time elapse.

Reproduce the fight for the lead: calculate and plot the difference of lap times between to two fighting cars at each lap will give people close look at the battle

The path to the championship: plot the position of each team at each season to see their Chronological performance

Does the significant change of regulations affect the results: For example, " In 2014, double points were awarded for the final race of the season to make it less likely that one dominant manufacturer or driver would build up an unassailable lead with several races still left, as had happened the year before". (List of Formula One World Championship points scoring system. from Wikipedia) The difference between 1st and 2nd driver and team will be calculated and the numbers before and after 2014 will be compared.

Interpretation will follow each modeling.

Challenges
----------

1.  Scrapping data from the website and download a lot of PDF files could be very time-consuming. I will investigate how to do the bulk downloading PDF file. Also, the JSON file from Ergast Developer API could be converted to data frame straightforwardly by fromJSON function in. I will do carefully study on the data structure.

2.  I will study how to create a free account the establish database at Amazon Relational Database Service (RDS) as I have not used it before. 

3.  The Formula One is complicated. Interpreting the data could be a challenge to me.
