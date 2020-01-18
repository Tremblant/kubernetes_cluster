SDGs
-----------------------

A brief illustratoion of exploratory data analysis(EDA) using data obtained from the World Bank website. This data shows the percentage of electricity coverage in the Sub-Saharan Africa and can be obtained [here](https://databank.worldbank.org/source/sustainable-development-goals-(sdgs)). 
Follow the above link, select Sub-Saharan Africa under `country`, under `series` check: Access to electricity(% of population), Access to electricity, rural(% of rural population) and Access to electricity, urban(% of urban population), and then under `time` check between 2012 to 2018 inclusive. Then you can download its csv format.

Installation
----------------------

### Download the data

* Clone this repo to your computer.
* Get into the folder using `cd SDGS`.
* Run `mkdir data`.
* Switch into the `data` directory using `cd data`.
* Download the data files from the World Bank into the `data` directory.  
    * You can find the data [here](https://databank.worldbank.org/source/sustainable-development-goals-(sdgs)).
* Extract all of the `.zip` files you downloaded.
    * On OSX, you can run `find ./ -name \*.zip -exec unzip {} \;`.
    * At the end, you should have two of csv files called `electricity.csv`, and `electricity_metadata.csv`.
* Remove all the zip files by running `rm *.zip`.
* Switch back into the `SDGS` directory using `cd ..`.

### Install the requirements
 
* Install the requirements using `pip install -r requirements.txt`.
    * Make sure you use Python 3.
    * You may want to use a virtual environment for this.

Usage
-----------------------

* Run `Data_Preparation.ipynb` in jupyter notebook to view the basic EDA performed.


Extending this
-------------------------

If you want to extend this work, here are a few places to start:

* You can perform more data visualization.
* Using more data from The World Bank you can find out how Sub-Saharan Africa compares to the rest of world in terms of electricity coverage and research on areas it should improve to attain vision 2030 under sustainable development goals.
* A lot of insights can be drawn from above notebook which can help create a platform for research. For example:

    * In the data visualization we see all the variables have an upward trend, though from 2012 upto around 2015 the electricity coverage    seems to be relatively flat. It will be interesting to find out why years between 2012 and 2015 have sluggish electricity coverage.
    
    * Comparing urban and rural electricity coverage, urban has a consistent upward trend while for rural it had a slight increase between 2012 and 2013, it then had a downward trend between 2013 and 2015 before shooting up again. This is also an interesting area of research, trying to find out why electricity coverage for rural behaves so and factors behind it.