import pandas as pd
from flask import Flask, render_template, request
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

app = Flask(__name__)


movies_df = pd.read_csv('movies.csv') 
movies_df.columns = movies_df.columns.str.strip()  

def compute_cosine_similarity():
    tfidf = TfidfVectorizer(stop_words='english')
    tfidf_matrix = tfidf.fit_transform(movies_df['genres'])
    cosine_sim = cosine_similarity(tfidf_matrix, tfidf_matrix)
    return cosine_sim


cosine_sim = compute_cosine_similarity()


def get_recommendations(movie_title, cosine_sim):
    try:
      
        idx = movies_df.index[movies_df['movie_title'].str.contains(movie_title, case=False, na=False)].tolist()
        if not idx:
            return []  

        idx = idx[0] 
        sim_scores = list(enumerate(cosine_sim[idx]))
        
 
        sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)[1:6]
        movie_indices = [i[0] for i in sim_scores]

        recommended_movies = movies_df['movie_title'].iloc[movie_indices].tolist()
        return recommended_movies
    except Exception as e:
        print(f"Error: {e}")
        return []  
    
@app.route("/", methods=["GET", "POST"])
def index():
    recommended_movies = []

    if request.method == "POST":
        movie_title = request.form.get("movie_title")
        if movie_title:
            recommended_movies = get_recommendations(movie_title, cosine_sim)

    return render_template("index.html", recommended_movies=recommended_movies)

if __name__ == "__main__":
    app.run(debug=True)

#front end code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Movie Recommendation System</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
        }
        .container {
            width: 80%;
            margin: 50px auto;
            background-color: #ffffff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }
        h1 {
            text-align: center;
            color: #333;
        }
        form {
            text-align: center;
            margin-bottom: 30px;
        }
        input[type="text"] {
            padding: 10px;
            font-size: 16px;
            width: 60%;
            margin-right: 10px;
            border-radius: 5px;
            border: 1px solid #ccc;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
        }
        button:hover {
            background-color: #45a049;
        }
        h2 {
            text-align: center;
            color: #333;
        }
        ul {
            list-style-type: none;
            padding: 0;
        }
        li {
            background-color: #e4e4e4;
            margin: 5px 0;
            padding: 10px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Movie Recommendation System</h1>
        <form method="POST">
            <label for="movie_title">Enter Movie Title:</label>
            <input type="text" id="movie_title" name="movie_title" required>
            <button type="submit">Get Recommendations</button>
        </form>

        {% if recommended_movies %}
            <h2>Recommended Movies:</h2>
            <ul>
                {% for movie in recommended_movies %}
                    <li>{{ movie }}</li>
                {% endfor %}
            </ul>
        {% elif recommended_movies is not none %}
            <h2>No recommendations found.</h2>
        {% endif %}
    </div>
</body>
</html>

  
