# NIL College Athletes Web Scrape


# Import statements (Requests, BeautifulSoup, regular expression, pandas and numpy)
import requests

from bs4 import BeautifulSoup

import re

import pandas as pd

import numpy as np


# Create empty lists to store athlete, sponsor, school, sport names, values and social media links

athlete_names = []
sponsor_names = []
school_names = []
sport_names = []
values = []
instagram_links = []
twitter_links = []


# Loop through the first 10 pages of the website and scrape athlete, sponsor, school, sport, and details link names.
# Range may be changed to scrape additional pages, up to 541.
for i in range(1, 11):
    url = f"https://nilcollegeathletes.com/deals?page={i}"
    html = requests.get(url)
    soup = BeautifulSoup(html.content, 'html.parser')

    # Find all the li tags with similar scraped data
    li_tags = soup.find_all('li', {'class': 'relative pl-0 pr-6 py-5 hover:bg-gray-50 sm:py-6'})

    # Loop through the <li> tags to find athlete, sponsor, school, sport, and social media links 
    for li in li_tags:
        find_athlete = li.find('a', href=re.compile('/athletes/([\w-]+)'))
        find_sponsor = li.find('a', href=re.compile('/sponsors/|/agencies/'))
        find_school = li.find('a', href=re.compile('/universities/'))
        find_sport = li.find('div', {'class': 'flex text-gray-500 text-sm space-x-2'})
        find_details = li.find('a', {'class': 'relative text-sm text-gray-500 hover:text-gray-900 font-medium'})

        if find_athlete is not None:
            athlete_url = 'https://nilcollegeathletes.com' + find_athlete['href']
            athlete_html = requests.get(athlete_url)
            athlete_soup = BeautifulSoup(athlete_html.content, 'html.parser')
            instagram_link = athlete_soup.find('a', href=re.compile('instagram.com/'))
            instagram_link = instagram_link.get('href') if instagram_link is not None else 'None'
            twitter_link = athlete_soup.find('a', href=re.compile('twitter.com/'))
            twitter_link = twitter_link.get('href') if twitter_link is not None else 'None'
        else:
            instagram_link = 'None'
            twitter_link = 'None'

        if find_sponsor is not None:
            sponsor_name = find_sponsor.text.strip()
        else:
            sponsor_name = 'None'

        if find_school is not None:
            school_name = re.findall(r'/universities/([^"]+)', str(find_school))[0]
            school_name = school_name.replace('-', ' ') # Replace the - symbols with spaces
        else:
            school_name = 'None'

        if find_sport is not None:
            sport_name = find_sport.find_all('span')[-1].text.strip()
        else:
            sport_name = 'None'

        value = np.nan  # Define value outside of the conditional block

        if find_details is not None:
            details = 'https://nilcollegeathletes.com' + find_details['href']
            # Follow the link in details and scrape the value
            html = requests.get(details)
            soup = BeautifulSoup(html.content, 'html.parser')
            value_tag = soup.find('dt', string='Value')
            if value_tag is not None:
                value = value_tag.find_next_sibling('dd').text.strip()
                value = re.findall(r'\$?\d[\d,.]*', value)
                value = value[0].replace(',', '').replace('$', '') if len(value) > 0 else 'NaN'
        else:
            value = 'NaN'

        athlete_names.append(find_athlete.text.strip())
        sponsor_names.append(sponsor_name)
        school_names.append(school_name)
        sport_names.append(sport_name)
        values.append(value)
        instagram_links.append(instagram_link)
        twitter_links.append(twitter_link)


# Create a DataFrame with athlete, sponsor, school, sport names, values and social media
nil = pd.DataFrame({'Athlete Name': athlete_names, 'Sponsor Name': sponsor_names, 'School': school_names, 'Sport': sport_names, 'Value': values, 'Instagram page': instagram_links, 'Twitter page': twitter_links})


print(nil)

![1](https://github.com/Cgrutsch/Final-Analytics-Project-Public/assets/123528826/e4149d28-d1ad-4925-a526-145b208df3ad)



sponsor_mode = nil['Sponsor Name'].mode()[0]
print(f"The most common occurring sponsor is: {sponsor_mode}")

  The most common occurring sponsor is: Barstool Sports
  
  
  
  
  
mean_value = nil['Value'].loc[nil['Value'] != 'NaN'].astype(float).mean()
print("Mean Value:", mean_value)

  Mean Value: 1916.0
  
# Query to search the dataFrame by athlete name
athlete_search = input("Enter athlete name: ")
athlete_data = nil[nil['Athlete Name'].str.contains(athlete_search, case=False)]
print(athlete_data)

Enter athlete name: JR Smith
   
![2](https://github.com/Cgrutsch/Final-Analytics-Project-Public/assets/123528826/cb59b067-c0ce-41ef-8784-547a5e27c971)


# Query to serach the dataFrame by school
school_search = input("Enter school name: ")
school_data = nil[nil['School'].str.contains(school_search, case=False)]
print(school_data)

Enter School Name: University of Nebraska Omaha

![3](https://github.com/Cgrutsch/Final-Analytics-Project-Public/assets/123528826/2333266e-57b3-45be-964b-2792e959017b)


# Query to search the dataFrame by sponsor
sponsor_search = input("Enter sponsor name: ")
sponsor_data = nil[nil['Sponsor Name'].str.contains(sponsor_search, case=False)]
print(sponsor_data)

Enter sponsor name: Gatorade
  
![4](https://github.com/Cgrutsch/Final-Analytics-Project-Public/assets/123528826/536ee8d9-4a26-4e76-8323-1fa296a09559)


# Query to search the dataFrame by sport
sport_search = input("Enter sport name: ")
sport_data = nil[nil['Sport'].str.contains(sport_search, case=False)]
print(sport_data)

Enter sport name: Golf
  
![5](https://github.com/Cgrutsch/Final-Analytics-Project-Public/assets/123528826/35b05f9f-a0c9-45c9-8371-ac91d5dea655)


# Import the Plotly express module to allow for figures to be generated 
import plotly.express as px


# Group by sport and count the number of occurrences
sport_count = nil.groupby('Sport').size().reset_index(name='Counts')


fig = px.pie(sport_count, values='Counts', names='Sport', title='NIL Deals by Sport')


fig.show()


![6](https://github.com/Cgrutsch/Final-Analytics-Project-Public/assets/123528826/4d33e994-a020-4d45-8bda-3ccd6b7ab4c0)
