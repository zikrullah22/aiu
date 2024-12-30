from flask import Flask, render_template_string, request
import os

app = Flask(__name__)

# Folder to save the uploaded files
UPLOAD_FOLDER = 'uploads'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# Ensure the upload folder exists
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

# HTML content combined with Flask backend logic
html_content = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Student Career Center</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }
        header {
            background-color: #4CAF50;
            color: white;
            padding: 10px 0;
            text-align: center;
        }
        section {
            margin: 20px;
        }
        footer {
            text-align: center;
            padding: 10px;
            background-color: #333;
            color: white;
        }
        textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            margin-bottom: 10px;
        }
        input[type="file"] {
            margin-bottom: 10px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px 20px;
            cursor: pointer;
        }
        button:hover {
            background-color: #45a049;
        }
    </style>
</head>
<body>

    <header>
        <h1>Welcome to the Career Center</h1>
        <p>Post your stories, photos, and videos to share with the university community!</p>
    </header>

    {% if not story_text %}
    <section id="story-post">
        <h2>Share Your Story</h2>
        <form action="/submit_story" method="POST" enctype="multipart/form-data">
            <label for="storyText">Your Story:</label><br>
            <textarea id="storyText" name="storyText" rows="4" placeholder="Write your story here..."></textarea><br>

            <label for="imageUpload">Upload an Image:</label><br>
            <input type="file" id="imageUpload" name="imageUpload" accept="image/*"><br><br>

            <label for="videoUpload">Upload a Video:</label><br>
            <input type="file" id="videoUpload" name="videoUpload" accept="video/*"><br><br>

            <button type="submit">Post Story</button>
        </form>
    </section>
    {% else %}
    <section id="story-submitted">
        <h2>Your Story has been successfully posted!</h2>
        <p><strong>Story Text:</strong> {{ story_text }}</p>
        
        {% if image %}
        <p><strong>Image Uploaded:</strong> <img src="{{ url_for('static', filename='uploads/' + image) }}" width="300" /></p>
        {% endif %}

        {% if video %}
        <p><strong>Video Uploaded:</strong> <video width="300" controls><source src="{{ url_for('static', filename='uploads/' + video) }}" type="video/mp4"></video></p>
        {% endif %}
        
        <a href="/">Go Back to Submit Another Story</a>
    </section>
    {% endif %}

    <footer>
        <p>&copy; 2024 Career Center, University Name</p>
    </footer>

</body>
</html>
"""

@app.route('/')
def index():
    return render_template_string(html_content, story_text=None)

@app.route('/submit_story', methods=['POST'])
def submit_story():
    story_text = request.form['storyText']
    
    # Handle image upload
    image = request.files.get('imageUpload')
    video = request.files.get('videoUpload')
    
    # Save the files if they exist
    if image:
        image.save(os.path.join(app.config['UPLOAD_FOLDER'], image.filename))
    if video:
        video.save(os.path.join(app.config['UPLOAD_FOLDER'], video.filename))

    return render_template_string(html_content, story_text=story_text, image=image.filename if image else None, video=video.filename if video else None)

if __name__ == '__main__':
    app.run(debug=True)
