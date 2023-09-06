# Flask File Upload with Nginx Reverse Proxy
A demonstration of file uploads using Flask and Nginx reverse proxy with `client_max_body_size` configuration.

```
High-level overview of the flow between NGINX, Flask (Python), and your web application for handling file uploads and demonstrating the client_max_body_size directive in action:

    User Interaction:
        A user accesses your website through their web browser.

    NGINX as Reverse Proxy:
        NGINX serves as a reverse proxy server that listens for incoming HTTP requests from users.

    Request Routing:
        When a user makes a request (e.g., uploads a file), NGINX routes the request to the appropriate location based on the URL. In your case, requests to the /upload path are directed to your Flask application.

    Flask Application (Python):
        NGINX forwards the request to your Flask application running on the same server or another server, depending on your configuration.
        Your Flask application handles the request. Specifically, the /upload route is responsible for processing file uploads.

    File Upload Handling:
        When a file is uploaded from the user's browser, it's sent as part of the HTTP POST request to the /upload route of your Flask application.
        Flask processes the uploaded file using the request.files object and saves it to a specified location on the server's file system.

    client_max_body_size in NGINX:
        NGINX ensures that the client_max_body_size directive is applied to the incoming request.
        If the uploaded file size exceeds the configured limit (e.g., 50MB), NGINX returns an error response (e.g., "413 Request Entity Too Large") without forwarding the request to Flask.

    Response Back to User:
        Depending on the outcome of the file upload handling in Flask, a response is generated.
        Success: If the file upload is successful, Flask sends a response indicating success.
        Failure: If there are any issues with the upload, Flask sends an appropriate error response.

    NGINX Response to User:
        NGINX receives the response from Flask and forwards it back to the user's browser.

    User Feedback:
        The user's browser displays the response received from the server, whether it's a success message or an error message.
```
        
This is the step-by-step guide to set up the Nginx configuration, Flask application, and server environment to allow file uploads and demonstrate the client_max_body_size in action.
For this guide, we'll assume you're using an Ubuntu-based system.

## Step 1: Install Required Software

- Install Nginx:

```
sudo apt update
sudo apt install nginx
```

- Install Python and Flask:
```
sudo apt install python3 python3-pip
pip3 install Flask
```

## Step 2: Set Up Flask Application

- Create a Directory for Your Flask App
- Choose a suitable location, like your home directory:
```
mkdir ~/myflaskapp
cd ~/myflaskapp
```

- Create a Flask App Script:
- Create a file named app.py using a text editor of your choice (e.g., nano, vim):

```
nano app.py
```

- Add the Flask Code (Paste the code contents from app.py)

```
from flask import Flask, request, render_template
import os

app = Flask(__name__)

UPLOAD_FOLDER = 'uploads'
if not os.path.exists(UPLOAD_FOLDER):
    os.makedirs(UPLOAD_FOLDER)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        uploaded_file = request.files['file']
        if uploaded_file:
            filename = uploaded_file.filename
            filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
            uploaded_file.save(filepath)
            return 'File uploaded successfully.'
    return render_template('upload_form.html')

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```


- Create HTML Template for Upload Form:
- Create a folder named templates in your Flask app directory (~/myflaskapp) and create a file named upload_form.html inside it. Add the following HTML code to upload_form.html:

```
    <!DOCTYPE html>
    <html>
    <body>

    <h2>File Upload Form</h2>

    <form action="/" method="POST" enctype="multipart/form-data">
        <input type="file" name="file">
        <input type="submit" value="Upload">
    </form>

    </body>
    </html>
```

## Step 3: Configure Nginx

- Create Nginx Configuration File:
- Create a new Nginx configuration file using a text editor:

Add Nginx Configuration:
Paste the following Nginx configuration into the file (myflaskapp):

```
server {
    listen 80;
    server_name your_domain.com; # Replace with your domain or server IP

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /uploads/ {
        alias /home/ubuntu/myflaskapp/uploads/; # Replace with your actual path
        internal;
    }
}
```

- Enable the Nginx Configuration and Restart Nginx:
```
    sudo nginx -t # Test the configuration
    sudo systemctl restart nginx
```

## Step 4: Test the Configuration

- Start the Flask App:
- In your ~/myflaskapp directory, run:
```
python3 app.py
```
- Access Your Website:    Open a web browser and navigate to http://your_domain.com or http://your_server_ip. You should see the file upload form.
- Upload a File:    Use the form to select a file and click "Upload." You should see a message indicating a successful upload.

## Step 5: Demonstrate client_max_body_size in Action

- Edit Nginx Configuration and Open the Nginx configuration file again:
- Add client_max_body_size Directive:
- Update the Nginx configuration to set a higher maximum upload size. Add the client_max_body_size directive under the location / block:
```
location / {
    proxy_pass http://127.0.0.1:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    client_max_body_size 50M; # Set the maximum upload size to 50MB
}
```
- Test the Upload: Restart Nginx to apply the new configuration:
```
    sudo systemctl restart nginx
```
   
  - Upload a Large File
  - Use the file upload form again, but this time, try uploading a file larger than 50MB. You should receive an error indicating that the file size exceeds the allowed limit.


Run the Flask application in the background using nohup:
```
nohup python3 app.py &
```
- nohup: Stands for "no hang up." It allows the application to continue running even after you close the terminal.
- &: Runs the command in the background.

    
That's it! You've set up the Nginx configuration, Flask application, and demonstrated the use of the client_max_body_size directive to control file upload sizes. Make sure to replace placeholders like your_domain.com, /path/to/your/uploads/folder, and other values with your actual information.
