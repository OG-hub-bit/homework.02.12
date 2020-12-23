import requests
import pprint
import datetime
from bs4 import BeautifulSoup




today = datetime.date.today()

base_url = 'https://kinoteatr.ru'
r = requests.get(f'{base_url}/raspisanie-kinoteatrov/')
soap = BeautifulSoup(r.text, 'lxml')
cinemas = []
for i in soap.findAll('div', class_='col-md-12 cinema_card'):
    name = i.find('h3').text.strip()
    href = i.find('a')['href'].strip()
    address = i.findAll('span', class_= 'sub_title')[0].text.strip()
    cinemas.append({
        'name': name,
        'href': href,
        'address': address
    })

    
shedule_dic = {}    
for i in range(7):
    date = today + datetime.timedelta(days=i)
    day = date.strftime('%d')
    month = date.strftime('%m')
    year = date.strftime('%y')
    

    
    allfilms = {}    
    for ind, cinema in enumerate(cinemas):
        r = requests.get(cinema['href'] + f"/?date={year}-{month}-{day}")
        soap = BeautifulSoup(r.text, 'lxml')
        films = soap.findAll('div', class_='shedule_movie bordered gtm_movie')
        films_dict = []
        for film in films:

            film_itself = {
                'name':film['data-gtm-list-item-filmname'],
                'href': film.find('a', class_='gtm-ec-list-item-movie')['href'],
                'format': film['data-gtm-format'],
                'genre': film['data-gtm-list-item-genre'],
                'raiting_sub': film.findAll('i', class_='raiting_sub')[0].text.strip(),
                "shedule": []
            }
            for time in film.findAll('a', class_="shedule_session"):
                film_itself["shedule"].append(time.find('span', class_='shedule_session_time').text.strip())

            

            films_dict.append(film_itself)    
        allfilms[cinema['name']] = films_dict
    shedule_dic[f"20{year}-{month}-{day}"] = allfilms
    
    
