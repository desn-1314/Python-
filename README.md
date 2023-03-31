import requests
from bs4 import BeautifulSoup

def get_movie_info(movie_url):
    """
    从电影详情页中获取电影信息
    """
    response = requests.get(movie_url)
    soup = BeautifulSoup(response.text, 'html.parser')

    # 获取电影标题
    movie_title = soup.find('span', attrs={'property': 'v:itemreviewed'}).text

    # 获取电影评分
    movie_rating = soup.find('strong', attrs={'class': 'rating_num'}).text

    # 获取电影评价人数
    movie_rating_num = soup.find('span', attrs={'property': 'v:votes'}).text

    # 获取电影导演
    movie_directors = soup.find_all('a', attrs={'rel': 'v:directedBy'})
    movie_directors = [director.text for director in movie_directors]

    # 获取电影演员
    movie_actors = soup.find_all('a', attrs={'rel': 'v:starring'})
    movie_actors = [actor.text for actor in movie_actors]

    # 获取电影类型
    movie_genres = soup.find_all('span', attrs={'property': 'v:genre'})
    movie_genres = [genre.text for genre in movie_genres]

    # 获取电影上映年份
    movie_year = soup.find('span', attrs={'class': 'year'}).text.strip('()')

    # 构造电影信息字典
    movie_info = {
        'title': movie_title,
        'rating': movie_rating,
        'rating_num': movie_rating_num,
        'directors': movie_directors,
        'actors': movie_actors,
        'genres': movie_genres,
        'year': movie_year
    }

    return movie_info

def get_top250_movies():
    """
    获取豆瓣电影 Top250 的电影列表
    """
    base_url = 'https://movie.douban.com/top250'
    movie_list = []

    for start in range(0, 250, 25):
        url = f'{base_url}?start={start}'
        response = requests.get(url)
        soup = BeautifulSoup(response.text, 'html.parser')

        # 获取电影列表
        movie_tags = soup.find_all('div', attrs={'class': 'hd'})
        movie_urls = [tag.a.get('href') for tag in movie_tags]

        # 获取电影信息
        for movie_url in movie_urls:
            movie_info = get_movie_info(movie_url)
            movie_list.append(movie_info)

    return movie_list

if __name__ == '__main__':
    movie_list = get_top250_movies()
    for movie in movie_list:
        print(movie)
