# Python - API Data Challenge
> How do I collect data and *definitively* answer questions? Specifically... "What is the weather like as we approach the equator (but with concrete data to back up our claims)?"
## Folder Contents
- Two `.ipynb` files containing all the data I processed, charts and graphs I generated, and any analyses I wrote. Anything insightful and analytical can be found there. The links you see upon opening the file are internal links that lead to other sections of the Notebook. It doesn't quite work on Github's preview system, but it should work if you fork it and open it with Jupyter or VSCode.
  - `WeatherPy` randomly generates unique cities around the world, collects weather data from OpenWeather, and plots the relationship between some of the weather variables on different scatter plots. Linear Regression graphs are also generated to show the strength of these relationships.
  - `VacationPy` filters through the data from the previous file, calls GeoApify to get the closest hotels of each city, and plots them all on a map through hvPlot.
- A .gitignore file that ignores common things like PyCache, Jupyter Notebook checkpoints, and, most importantly, API Keys. 
- A folder with
  - A `.csv` with the randomly generated cities I used for the displays shown in these Notebook files. I don't think I'm able to get the random seed here, so here's the next best thing with the cities I generated.
  - Pictures of the scatter plots generated from the data retrieved based on the cities generated in that `.csv`. Each one showcases a relationship between the latitude of a city and some other weather parameter.

### Installation/Prerequisites
- Make sure you can run Python. The development environment I used was set-up with:
```
conda create -n dev python=3.10 anaconda -y
```
#### Imported Modules
- Installing via the conda command given should give you access to all of the script's modules locally. However, if you don't have them, be sure to grab yourself the following libraries:
  - [Pandas](https://pandas.pydata.org/docs/getting_started/install.html), [NumPy](https://numpy.org/install/), [SciPy](https://scipy.org/install/), for your basic dataframe and statistical manipulation.
  - [Requests](https://requests.readthedocs.io/en/latest/), [hvPlot](https://hvplot.holoviz.org/getting_started/installation.html), [GeoViews](https://geoviews.org/), and [CitiPy]([https://matplotlib.org/stable/users/installing/index.html](https://github.com/wingchen/citipy)) for all the geographical information and city stuff to be done.
#### API Keys
- You'll also need to get **2 API Keys**:
  - For the WeatherPy script, you'll need to get one for [OpenWeather](https://openweathermap.org/api), which needs you to make an account for 1,000 free daily calls.
  - For the VacationPy script, you'll need to make an account and get an API key from [GeoApify](https://apidocs.geoapify.com/)
- Assuming that you don't want to change any of the code, you'll then need to create a python file called `api_keys.py` and assign your keys under the variables `weather_api_key` and `geoapify_key` in this format:
```python
weather_api_key = "a1b2c3d4e5f6g7h8i9"
```

## Code Showcase
The purpose of this assignment is really to get us accustomed to reading documentation and figuring out how to work with APIs.

Really, there are 2 main challenges here, especially with something like OpenWeather where you only have so many daily calls to make before they start charging you or blocking you from further testing:
1. Figuring out what to order in your GET request.
2. Figuring out what's in your order and how to access the parts you want.

As previously mentioned, all the insightful stuff you need are in the comments of the attached `.ipynb` files, so I would like to instead talk about a few snippets of code I'm particularly fond of and maybe explain my process behind them.

### Tight Budget...
As I've previously mentioned, OpenWeather only allows for 1,000 calls per day. This may seem plenty, but WeatherPy does need to make around 600 separate calls for the cities it generates. This means that you can't even test each bunch of cities before running out of calls. You could make a bunch of dummy accounts and get backup Keys that way, but that's tedious and a reckless waste of time, so here are my two guidelines:
- "When in doubt, print it out." - The console is your friend and you can easily visualize what you've returned through it.
- "Read the docs, get one done." - Follow the rules of the documentation and try to nail one request perfectly. Testing 1 result consumes drastically less of your call budget than testing 600. Once you get one down, all you need to do for 600 is to throw it into a loop.

As with any game, the first thing to do is the read the rule books.
- [GeoApify (Places) Docs](https://apidocs.geoapify.com/docs/places/#about)
- [OpenWeather (Current Data) Docs](https://openweathermap.org/current)

Upon reading the documentation for OpenWeather, you'll find that a sample response for a given call would look something like this (in JSON) form:
```python
{
  "coord": {
    "lon": 10.99,
    "lat": 44.34
  },
  "weather": [
    {
      "id": 501,
      "main": "Rain",
      "description": "moderate rain",
      "icon": "10d"
    }
  ],
  "base": "stations",
  "main": {
    "temp": 298.48,
    "feels_like": 298.74,
    "temp_min": 297.56,
    "temp_max": 300.05,
    "pressure": 1015,
    "humidity": 64,
    "sea_level": 1015,
    "grnd_level": 933
  },
  # ...
  # ... A lot more data down here.
```
All we need to do then is to pass in something like this to file a GET request in order to access that information:
```python
https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={API key} #&otherParameters=otherValues here
```
Since the responses of our requests are all JSON objects, we can simply access each response by navigating through it like you would a dictionary of dictionaries and get the things we need. For example, we can get the humidity of a JSONified response with:
```python
response["main"]["humidity"]
```
Through slight trial and error, once the parameters we need have been confirmed and once we know how to get all the information we need, then we can loop through everything and throw the results into a Pandas Dataframe.

The rest is just Pandas, NumPy, and MatPlotLib things we've already learned, and we can figure out the customization through their docs as well to produce something like:

![A linear regression plot](https://cdn.discordapp.com/attachments/1107347677831778364/1110352314335764500/image.png)

- You can find out how I got there by cloning this repository, but the short answer is: Googling Documentation.

### Documentation that's harder to Google
A problem I find with some tutorials and guides including as my own is that we have an "Owl Problem". Picture a tutorial that teaches you how to draw an owl:
```
How to draw an owl:
Step 1. Draw a circle
Step 2. Draw the rest of the owl
```
It's frustrating to see people take the easy steps for granted even if some of the steps do look incredibly obvious in hindsight. HVPlot documentation, I find, was a little hard to navigate, so I thought it would be nice if I showed you what I looked at to get the code I needed.
1. Firstly, look up the [official documentation](https://hvplot.holoviz.org/). At a first glance, without even scrolling down, the documentation says:
> `.hvplot()` is a powerful and interactive Pandas-like `.plot()` API. By replacing `.plot()` with `.hvplot()` you get an interactive figure.
2. If you know Pandas, you can simply look up Pandas documentation and figure out the rest from there through pattern recognition and experience. However, if you aren't so confident, you can notice their examples and find that the general pattern seems to be `yourDataFrame.hvplot.someFunction(x="dataframe column 1", y="dataframe column 2", otherKeyword="other value")`
3. Following that logic, we can find out *what* we can customize and *how* we can do so by either looking directly in the documentation or by Googling "hvplot customize (this thing)" and clicking on links such as these:
    - [Customization Options](https://hvplot.holoviz.org/user_guide/Customization.html). With a quick scroll or some quick Ctrl+F searches, we can find stuff such as `geo=True` to turn our x (longitude) and y (latitude) coordinates into map coordinates for geographical representation.
    - [Points](https://hvplot.holoviz.org/reference/geopandas/points.html). We can find out how to customize each point here through this page and the customization options from the previous link.
    - Following the same pattern, we can Google and repeat the process until the puzzle fits together. e.g.: I want to find out tile options for hvplot. I can Google that and find [this page here](https://holoviews.org/reference/elements/bokeh/Tiles.html) as part of hvPlot's various dependencies.
4. If that still isn't enough, you could always grab StackOverview answers or find tutorials on Youtube if you have the time to spare.

A lot of coding is just knowing how to find that syntax. Your brain can't compare with the collective intelligence of the internet, so the next best course of action is to learn from where in the internet to ~~steal~~ acquire the information you need. If I can do it, so can you. I believe in you.

## Resources that helped a lot
Assuming you reviewed all your Pandas content, really all the answers are in your API documentation. However, here are a few videos that can ease you into the testing process:
- [API basics](https://www.youtube.com/watch?v=ChwSD5e0Qs0) - Every API has their own parameters and query patterns, but they all follow similar enough rules for knowledge of one API to carry over to the next. If you know how a request works, how to formulate a request URL, how to use the Requests library, and how to wrangle the data collected from a request, you're golden. All you need from there is patience, basic reading comprehension skills, and practice.
> Time, documentation, and practice takes you a long way for coding in general, now that I think about it.

- [Try/Except](https://www.youtube.com/watch?v=j_q6NGOwDJo) - The API isn't going to have all your answers, so it's a good idea to have a backup plan to keep the work going with [Exception Handling](https://www.w3schools.com/python/gloss_python_error_handling.asp) to save yourself from painstakingly cleaning out all the calls that don't have the things you want.

## Random Thoughts
> Project completed on May 22, 2023
- A while ago, there was a project about [tracking rich people and their private jets](https://blog.apilayer.com/how-a-college-student-was-able-to-track-elon-musks-private-jet-movements-by-simply-using-a-few-apis/) that got popular enough to make people uncomfortable. Unfortunately, the Twitter Bot that would post live updates has been since banned, though the [website itself seems to still be running just fine](https://www.superyachtfan.com/private-jet/owner/elon-musk/).
![A picture of what the private jet tracker looks like](https://cdn.discordapp.com/attachments/1107347677831778364/1110348647268356116/image.png)
- Given that aviation information is public, people like us could grab APIs on the web like [AviationStack](https://aviationstack.com/?utm_source=blog&utm_medium=blog&utm_campaign=NA) and reproduce that. We can make giants quiver in mild discomfort just by standing atop the shoulders of other giants.
- The hardest part of data science is arguably the collection of good data. APIs facilitate that so we don't have to spend months mind-numbingly doing data entry.
