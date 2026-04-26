[school_social_website_flask_app.py](https://github.com/user-attachments/files/27092952/school_social_website_flask_app.py)
from flask import Flask, render_template, request, redirect, url_for, session
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret123'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'

db = SQLAlchemy(app)

# Models
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(100), unique=True)
    password = db.Column(db.String(200))

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.Text)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))

# Routes
@app.route('/')
def home():
    posts = Post.query.all()
    return render_template('home.html', posts=posts)

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form['username']
        password = generate_password_hash(request.form['password'])
        user = User(username=username, password=password)
        db.session.add(user)
        db.session.commit()
        return redirect(url_for('login'))
    return render_template('register.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        user = User.query.filter_by(username=request.form['username']).first()
        if user and check_password_hash(user.password, request.form['password']):
            session['user_id'] = user.id
            return redirect(url_for('home'))
    return render_template('login.html')

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('home'))

@app.route('/post', methods=['POST'])
def post():
    if 'user_id' in session:
        content = request.form['content']
        new_post = Post(content=content, user_id=session['user_id'])
        db.session.add(new_post)
        db.session.commit()
    return redirect(url_for('home'))

if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run(debug=True)

# Templates (create a templates folder and add these files)

# home.html
"""
<!DOCTYPE html>
<html>
<head>
<title>School Feed</title>
</head>
<body>
<h1>School Feed</h1>

{% if session.user_id %}
<form method="POST" action="/post">
<textarea name="content" placeholder="Write something..."></textarea>
<button type="submit">Post</button>
</form>
<a href="/logout">Logout</a>
{% else %}
<a href="/login">Login</a> | <a href="/register">Register</a>
{% endif %}

<hr>

{% for post in posts %}
<p>{{ post.content }}</p>
<hr>
{% endfor %}

</body>
</html>
"""

# login.html
"""
<form method="POST">
<input name="username" placeholder="Username">
<input name="password" type="password" placeholder="Password">
<button type="submit">Login</button>
</form>
"""

# register.html
"""
<form method="POST">
<input name="username" placeholder="Username">
<input name="password" type="password" placeholder="Password">
<button type="submit">Register</button>
</form>
"""
