# Spotify-most-streamed-songs

Dashboard Link - [https://app.powerbi.com/groups/me/reports/2fb98590-42f4-4b51-be44-6c55404ed16f/b6e7faa788c2079b6b27?experience=power-bi&clientSideAuth=0](https://app.powerbi.com/groups/me/reports/275c6349-e265-45ad-8728-5b65557c3416/9fbe4f27d0e30109b7ae?experience=power-bi)

### About Project

This project is built using Power BI, Powerpoint, Chat GPT, Python, HTML, 3rd party Visual called HTML content and Deneb


### Steps followed 

- Step 1 : With the help of chatgpt below python script was used to fetch cover images of the album and add the image URL in a new column [API credentials was replaced]

```python
import requests
import pandas as pd
import time

# Your Spotify API credentials
CLIENT_ID = 'your_client_id_here'
CLIENT_SECRET = 'your_client_secret_here'

# Get access token
auth_url = 'https://accounts.spotify.com/api/token'
auth_response = requests.post(auth_url, {
    'grant_type': 'client_credentials',
    'client_id': CLIENT_ID,
    'client_secret': CLIENT_SECRET,
})

auth_response_data = auth_response.json()
access_token = auth_response_data['access_token']
headers = {'Authorization': f'Bearer {access_token}'}

# Load your dataset
df = pd.read_excel("Onyx Data DataDNA Datatset Challenge - Spotify Most Streamed Songs 2023 Dataset - October 2023.xlsx", sheet_name="Spotify Dataset")

# Function to search and get cover URL
def get_cover_url(track, artist):
    query = f'track:{track} artist:{artist}'
    search_url = f"https://api.spotify.com/v1/search?q={requests.utils.quote(query)}&type=track&limit=1"
    response = requests.get(search_url, headers=headers)
    data = response.json()
    try:
        return data['tracks']['items'][0]['album']['images'][0]['url']
    except (IndexError, KeyError):
        return "Not Found"

# Apply function
df['spotify_cover_url'] = df.apply(lambda row: get_cover_url(row['track_name'], row['artist(s)_name']), axis=1)

# Save to new file
df.to_excel("spotify_dataset_with_covers.xlsx", index=False)
```

- Step 2 : The newly generated excel dataset was loaded into Power BI Desktop.
- Step 3 : Open power query editor & in view tab under Data preview section, check "column distribution", "column quality" & "column profile" options.
- Step 4 : Also since by default, profile will be opened only for 1000 rows so you need to select "column profiling based on entire dataset" to check the empty and error percentage.
- Step 5 : It was observed that in none of the columns used for the dashboard had any errors or empty cells and hence the blanks were ignored
- Step 6 : below DAX query was used to create a new column using the release year, release month and release Date

```DAX
date = DATE(Sheet1[released_year],Sheet1[released_month],Sheet1[released_day])
```
- Step 7 : A Date table was created using beloe query and relationship was established between Date Table and Date column in sheet 1

```DAX
date = CALENDAR(MIN(Sheet1[date]),MAX(Sheet1[date]))
```
- Step 8 : Below DAX expressions were written for the KPI

Average stream per year
```DAX
Average stream per year = CALCULATE(AVERAGE(Sheet1[streams]),ALLEXCEPT(Sheet1,'date'[year]))
```

Max Stream
```DAX
Max Stream = MAX(Sheet1[streams])
```

Average Energy Percentage Value 
```DAX
Percentage Val = AVERAGE(Sheet1[energy_%])
```

Number of releases per year
```DAX
release by year = COUNT(Sheet1[track_name])
```

Stream count compared to avg stream value
```DAX
Top Song vs avg val = 
DIVIDE([Max Stream]-[Average stream per year],[Average stream per year])
```

Image display [ below html and css scriot was included in the DAX query and HTML content visual was used to display the Album cover]
Double quotes was replaced with single quotes in the HTML script

```DAX
Image URL = 
 var x =
CALCULATE(max(Sheet1[cover_url]),Sheet1[streams]=MAX(Sheet1[streams]))
return

"
<!DOCTYPE html>
<html lang='en'>
<head>
  <meta charset='UTF-8' />
  <meta name='viewport' content='width=device-width, initial-scale=1.0'/>
  <title>Fixed Image 16:9 with Rounded Corners</title>
  <style>
    body {
      margin: 0;
      padding-top: 60vh; /* Ensure content doesn't overlap image */
      font-family: sans-serif;
    }

    .image-container {
      position: fixed;
      top: 0;
      left: 0;
      bottom: 0;
      width: 100vw;
      overflow: hidden;
      border-radius: 10px 10px 10px 10px; /* Round bottom corners only */
      z-index: 999;
    }

    .image-container img {
      width: 100%;
      height: 330px;
      object-fit: cover;
      object-position: center;
      display: block;
      opacity:0.3;
    }

    .content {
      padding: 2rem;
    }
  </style>
</head>
<body>

  <div class='image-container'>
    <img src='"&x&"' alt='Album Cover'>
  </div>
</body>
</html>
"
```

Deneb Unit graph - https://tinyurl.com/yrchdb7c

```JSON
{
  "$schema": "https://vega.github.io/schema/vega/v5.json",
  "width": 320,
  "height": 320,
  "padding": 15,
  "signals": [
    {
      "name": "textGradient",
      "update": "{gradient: 'linear', stops: [{offset: 0, color: '#036d19'}, {offset: 1, color: '#1db954'}]}"
    },
    {
      "name": "percent",
      "update": "0",
      "on": [
        {
          "events": {
            "type": "timer",
            "throttle": 0
          },
          "update": "round(data('dataset')[0]['Percentage Val'])"
        }
      ]
    }
  ],
  "data": [
    {"name": "dataset"},
    {
      "name": "back",
      "values": [],
      "transform": [
        {
          "type": "sequence",
          "start": 0,
          "stop": 100,
          "step": 1,
          "as": "val"
        },
        {
          "type": "formula",
          "expr": "1",
          "as": "t"
        },
        {
          "type": "pie",
          "field": "t",
          "startAngle": {"signal": "0"},
          "endAngle": {"signal": "2*PI"}
        }
      ]
    },
    {
      "name": "front",
      "values": [],
      "transform": [
        {
          "type": "sequence",
          "start": 0,
          "stop": {"signal": "percent"},
          "step": 1,
          "as": "val"
        },
        {
          "type": "formula",
          "expr": "1",
          "as": "t"
        },
        {
          "type": "pie",
          "field": "t",
          "startAngle": {"signal": "0"},
          "endAngle": {
            "signal": "((2*PI)/100)*percent"
          }
        }
      ]
    }
  ],
  "scales": [
    {
      "name": "color",
      "type": "linear",
      "domain": {
        "data": "back",
        "field": "val"
      },
      "range": [
        "#036d19",
        "#1db954"
      ]
    }
  ],
  "marks": [
    {
      "type": "arc",
      "from": {"data": "back"},
      "encode": {
        "enter": {
          "fill": {"value": "#b3b3b3"},
          "x": {"signal": "width / 2"},
          "y": {"signal": "height / 2"}
        },
        "update": {
          "startAngle": {
            "field": "startAngle"
          },
          "endAngle": {
            "field": "endAngle"
          },
          "padAngle": {
            "signal": "0.015"
          },
          "innerRadius": {
            "signal": "(width / 2)-15"
          },
          "outerRadius": {
            "signal": "width / 2"
          }
        }
      }
    },
    {
      "type": "arc",
      "from": {"data": "front"},
      "encode": {
        "enter": {
          "fill": {
            "scale": "color",
            "field": "val"
          },
          "x": {"signal": "width / 2"},
          "y": {"signal": "height / 2"}
        },
        "update": {
          "startAngle": {
            "field": "startAngle"
          },
          "endAngle": {
            "field": "endAngle"
          },
          "padAngle": {
            "signal": "0.015"
          },
          "innerRadius": {
            "signal": "(width / 2)-15"
          },
          "outerRadius": {
            "signal": "width / 2"
          }
        }
      }
    },
    {
      "type": "arc",
      "data": [{"a": 1}],
      "encode": {
        "enter": {
          "fill": {"value": "#b3b3b3"},
          "x": {"signal": "width / 2"},
          "y": {"signal": "height / 2"}
        },
        "update": {
          "startAngle": {"signal": "0"},
          "endAngle": {
            "signal": "2*PI"
          },
          "innerRadius": {
            "signal": "(width / 2)-25"
          },
          "outerRadius": {
            "signal": "(width / 2)-20"
          }
        }
      }
    },
    {
      "type": "text",
      "data": [{}],
      "encode": {
        "update": {
          "text": {
            "signal": "percent + '%'"
          },
          "align": {"value": "center"},
          "fontWeight": {
            "value": "bold"
          },
          "fill": {
            "signal": "textGradient"
          },
          "x": {"signal": "width /2"},
          "y": {"signal": "width /2"},
          "dy": {"value": 10},
          "fontSize": {"value": 70}
        }
      }
    },
    {
      "type": "text",
      "data": [{}],
      "encode": {
        "update": {
          "text": {
            "value": "Energy Percentage"
          },
          "align": {"value": "center"},
          "fontWeight": {
            "value": "bold"
          },
          "fill": {"value": "#9092a1"},
          "x": {"signal": "width /2"},
          "y": {"signal": "width /2"},
          "dy": {"value": 40},
          "fontSize": {"value": 25}
        }
      }
    }
  ]
}
```

- Step 9 : Conditional formating was added for Deneb unit chart for the arc and outer radius based on the energy percentage value
- Step 10 : Conditional formating was added to show if the album has the number of streams greater than or lesser than the average stream
- Step 11 : New card visual was added to show the Track name, Artist, Release Name, number of artists, number of streams, Acousticness, Danceability, liveness, Speechness and Valence
- Step 12 : Line graph is used to show the trend of number of releases each year
- Step 13 : Slicer chart is used for Date, Artist, Track and year of release
  
# Snapshot of Dashboard (Power BI Service)

![image](https://github.com/user-attachments/assets/0c31df20-0f45-49e8-9fc4-1de9326c321c)

 
 # Report Snapshot (Power BI DESKTOP)


![image](https://github.com/user-attachments/assets/80390421-047d-4ad9-92a7-bdcc00083468)


## Author - Venkatesh Varma V

This project is part of my portfolio, showcasing my Power BI skills essential for Data Analyst roles.
