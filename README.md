# Spotify History

#### Dashboard Link: https://app.powerbi.com/view?r=eyJrIjoiMmEyYjQzYTYtMjUyOS00M2E5LWE2N2QtZjE2MmU1N2NhNmNhIiwidCI6IjIzODcwYmMyLTExMjQtNDJlNS05MzBmLTFlMmVhMjE4ZjcyNSJ9

## Statement

This is a personal project that showcases my streaming history from May 1st 2017 to February 6th 2026. I wanted to find additional ways to display my Spotify data as Spotify themselves does not provide users with many options to do so within their app. 

### The Process

Step 1: Upon opening the zip file that was sent after requesting the data, 19 JSON files were present each representing a certain amount of data per year.

Step 2: To focus on the entire history from start to finish, Python in VS Code was used to compile all files together into one JSON file making it easier to work with

Step 3: To load the JSON into Python the following lines of code were used:

    import json

    with open('Full_Streaming_History_2017-2026.json', 'r') as file:
        data = json.load(file)

Step 4: The data included many keys so Pandas, a software library for Python, was used to extract the keys needed and turned them into the column names. This was done with these lines of code:

    import pandas as pd

    table = pd.DataFrame(data=data, columns=['ts', 'platform', 'ms_played', 'master_metadata_track_name', 'master_metadata_album_artist_name', 'master_metadata_album_album_name', 'skipped'])

Description of each column:

'ts': Full date and time from when the track stopped playing
'platform': Device used for that stream
'ms_played': Amount of milliseconds played of the track
'master_metadata_track_name': Name of the track played
'master_metadata_album_artist_name': Name of the artist
'master_metadata_album_album_name': Name of the album
'skipped': Whether the track was skipped or not (true/false)

Step 5: The columns were renamed to make it easier to read using this line of code:

    table.rename(columns={'ts': 'Timestamp', 'ms_played': 'Playtime(ms)', ‘platform’: ‘Device Used’, ‘master_metadata_track_name’: ‘Track’, ‘master_metadata_album_artist_name’: ‘Artist’, ‘master_metadata_album_album_name’: ‘Album’, ‘skipped’: ‘Skipped’}, inplace=True)

Step 6: In the 'Device Used' column there were many instances of the same device being labeled differently. For ios it would list the ios version and in brackets state the device (iPhone or iPad) and the model. For Windows, it specifically stated whether you used the Windows app, the Spotify web player on the website, or it would just state the Windows version. Since this creates extra device categories for the same device, this line of code was used to create a new column:

    table["Device Used Clean"] = table["Device Used"].str.split().str[0]

This allowed for only "ios", "Android-tablet", "web_player", "windows" and "Windows" to be device categories in the new column

Step 7: To fix the values in the device column to either be iOS, Android or Windows the following lines of code were used:

    mapping = {
        "ios": "iOS",
        "Android-tablet": "Android",
        "web_player": "PC",
        "windows": "PC",
        "Windows": "PC"
    }

    table["Device Used Clean"].replace(mapping, inplace=True)

This line of code made it easy to map where those values were and what they should be. Then using replace(), the values were changed accordingly making for only 3 categories. The inplace=True allows for the change to be permanent so the data can continue to be manipulated with this cleaned column.

Step 8: The line of code below was used to make sure the column uniquely listed only iOS, Android, and Windows making sure the change went through.

    table["Device Used Clean"].unique().tolist() 

Step 9: Using the line of code below, the table was transformed into a CSV file to be further cleaned and displayed in Power BI.

    table.to_csv("Full Streaming History C.csv")

Step 10: Once the CSV was loaded into Power BI, power query editor was used to further transform the data.

Step 11: A column titled ‘Unnamed: 0’ was changed to ‘StreamID’.

Step 12: Since the ‘Timestamp’ column included both the date and time, the column was duplicated. The original became the date only column and the other was the time column. The columns were renamed accordingly and reordered so the date and time columns were next to each other.

Step 13: A new column was added using the time functions in Power Query M:

    = Table.AddColumn(#"Renamed Columns2", "Time of Day", each if Time.From([Time]) >= #time(5,0,0) and Time.From([Time]) < #time(11,59,0) then "Morning"
    else if Time.From([Time]) >= #time(12,0,0) and Time.From([Time]) < #time(16,59,0) then "Afternoon"
    else if Time.From([Time]) >= #time(17,0,0) and Time.From([Time]) < #time(21,0,0) then "Evening"
    else "Night")

This column called 'Time of Day' allowed for a more broad look at the general time of day the stream took place instead of using the time down to the exact second. 

Step 14: The 'Skipped' column was removed due to not wanting to include it in the report.

Step 15: The 'Device Used' column was removed and 'Device Used Clean' was renamed 'Device Used'.

Step 16: The 'Time' column was removed as having both the specific time and general time of day felt unneeded.

Step 17: Empty string values were discovered in the 'Album' column. Once located it was found that it was an album that correlated with one artist so the empty values were replaced with the album title 'NA'.

Step 18: An artist was listed as both 'NakJoon (Bernard Park)' and 'Bernard Park'. Values equal to 'NakJoon (Bernard Park)' were changed to 'Bernard Park'.

Step 19: Top Five Artists and Albums Page Creation
- A donut chart was created as a visual to show the top 5 artists by the sum of playtime. 
- Using bookmarks and a bookmark navigator, an additional donut chart for top 5 albums by sum of playtime can be shown when using the navigator. 
- A slicer visual is present to select the year interacting with the page.
- Six card visuals are used to show the top track, top track listens, artist, distinct tracks listened to by artist, total playtime and total tracks listened to by artist.
    - When interacting with the slicer, it includes all artists and when interacting with the chart the artist card visuals will show information corresponding to that artist. This applies to the album chart as well.

Step 20: Overview of Listening History Page Creation
- An area chart was used to display the sum of playtime each year.
- An additional area chart was created to display the sum of playtime each month.
- A slicer was created to make it easier to select the year of interest. 

Step 21: Time of Listening Page Creation
- A 100% stacked column chart was used to display the year and the amount of time of listening at a specific time of day measured by the sum of playtime. 
- Four card visuals were placed to interact with the chart showing the year, the time of day, the top track and the top artist.

Step 22: First and Last Listen Page Creation
- An area chart was created to show the sum of playtime by every month of each year. 
- A slicer visual has a dropdown with a search bar to select the artist of interest.
- Two card visuals were used to show the first time the artist appears and the last time they make an appearance in the streaming history.
    - Interaction with the area chart and slicer will have the card visuals display the first and last time you listened to the artist that month. Just the area chart is the first and last listen in general.
- A table visual was used to show the top tracks listened by that artist. 
    - When interacting with only the area chart, the table chart will display the top 5 tracks listened to within that month. With the slicer it will specify the songs by the chosen artist.

Step 23: Device Preference Page Creation 
- An area chart was created to show which devices were used the most each year measured by the sum of playtime. 
- An additional area chart was created showing only the months allowing for a monthly comparison depending on the selected year.
- A slicer was added for quarter allowing for the ability to compare the months within quarters instead of viewing all 12.
    - This interacts with the area chart for year as well only basing the sum of playtime with the selected months of that quarter.

Step 24: Summary Page Creation
- This page includes four of the chart visuals with the top five artists, playtime by year, time of day and device used by year.
- There are three card visuals representing the first stream, last stream and artist.
- The last visual is a slicer used for selecting the year interacting with all visuals.

Step 25: The report was then published to Power BI Service.

# Insights

### About 328,390 tracks were listened to since the start of the recorded Spotify history with 9260 being distinct tracks and a sum of 52 billion milliseconds in playtime.
The top track by sum of playtime is Dynamite by BTS with 6433 listens.
- The top artist by playtime is BTS with 13 billion milliseconds making up for 25% of the total playtime across all years.

###  Out of the top five albums listened to in 2024, Clancy by Twenty One Pilots makes up for 49.78% of playtime while the other four albums share the other 50.22%.

### 2021 had the highest amount of playtime being 17.8 billion milliseconds which is equivalent to 6 months, 3 weeks, 4 days, 19 hours, 31 minutes and 12 seconds.
Peak month of listening in 2021 was in April at 1.6 billion milliseconds making up for almost 9% of the total playtime in 2021.
- The year with the lowest amount of playtime was 2019 with 1 billion milliseconds or 1 week, 4 days, 13 hours, 46 minutes and 40 seconds.

***This only includes years that have streams across all months. 2017 and 2026 are excluded from this.***

### 2024 was the closest to an even split across all periods of the day with the total percentage of playtime being almost 25% each.
The afternoon is overall the preferred time of day for music listening.

### The overall top artist by playtime, BTS, was first streamed January 24th, 2018.
BTS peak playtime was in August 2020 with about 989 million milliseconds of playtime.

### PC was the most preferred device for listening from 2020-2023 with a peak of 17.8 billion milliseconds of playtime.
Android was overall the less preferred device with its first appearance in December of 2017 at 151 million milliseconds and its second in December of 2019 with only 187378 milliseconds of playtime. 

### In summary the Spotify data reflects a change in streaming especially during 2020-2022 which corresponds with when COVID-19 hit. 
This is a possible reason as to why the playtime was highest during these years and why a dropoff is seen afterwards as schools reopened.
