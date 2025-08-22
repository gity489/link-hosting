<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>File Sharing Website</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            background-color: #f5f5f5;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        .container {
            background-color: #fff;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.1);
            width: 80%;
            max-width: 600px;
        }

        h1 {
            text-align: center;
            color: #333;
        }

        input[type="text"] {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ccc;
            border-radius: 5px;
        }

        input[type="file"] {
            display: block;
            margin: 20px 0;
        }

        button {
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        button:hover {
            background-color: #45a049;
        }

        ul {
            list-style-type: none;
            padding: 0;
        }

        li {
            background-color: #f9f9f9;
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 5px;
        }

        li a {
            text-decoration: none;
            color: #333;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Your File Sharing Website</h1>

        <!-- Search Bar -->
        <input type="text" id="search-bar" placeholder="Search your links, images, and videos..." onkeyup="searchFiles()">

        <!-- Upload Form -->
        <form id="upload-form" enctype="multipart/form-data">
            <label for="file-upload">Upload File (Max 1500 KB)</label>
            <input type="file" id="file-upload" name="file" accept="image/*,video/*,text/*" required>
            <button type="submit">Upload</button>
        </form>

        <!-- Display Uploaded Files -->
        <h2>Your Uploads</h2>
        <ul id="file-list">
            <!-- Uploaded files will appear here -->
        </ul>
    </div>

    <script>
        // Handle Search
        function searchFiles() {
            const query = document.getElementById("search-bar").value.toLowerCase();
            const files = document.querySelectorAll("#file-list li");

            files.forEach(file => {
                const fileName = file.textContent.toLowerCase();
                if (fileName.includes(query)) {
                    file.style.display = "";
                } else {
                    file.style.display = "none";
                }
            });
        }

        // Handle Upload
        const uploadForm = document.getElementById("upload-form");
        uploadForm.addEventListener("submit", function(e) {
            e.preventDefault();

            const formData = new FormData(uploadForm);
            fetch("/upload", {
                method: "POST",
                body: formData,
            })
            .then(response => response.json())
            .then(data => {
                if (data.success) {
                    loadUploadedFiles();
                } else {
                    alert("Upload failed!");
                }
            })
            .catch(err => alert("Error: " + err));
        });

        // Load Uploaded Files
        function loadUploadedFiles() {
            fetch("/files")
            .then(response => response.json())
            .then(files => {
                const fileList = document.getElementById("file-list");
                fileList.innerHTML = "";

                files.forEach(file => {
                    const listItem = document.createElement("li");
                    listItem.innerHTML = `<a href="${file.url}" target="_blank">${file.name}</a>`;
                    fileList.appendChild(listItem);
                });
            });
        }

        // Initial load
        loadUploadedFiles();
    </script>

    <script>
        const express = require("express");
        const multer = require("multer");
        const path = require("path");
        const fs = require("fs");

        const app = express();
        const PORT = 3000;

        // Create a directory for file uploads if it doesn't exist
        const uploadDir = path.join(__dirname, "uploads");
        if (!fs.existsSync(uploadDir)) {
            fs.mkdirSync(uploadDir);
        }

        // Set up multer storage with a size limit of 1500 KB
        const storage = multer.diskStorage({
            destination: (req, file, cb) => {
                cb(null, uploadDir);
            },
            filename: (req, file, cb) => {
                cb(null, Date.now() + path.extname(file.originalname));
            },
        });

        const upload = multer({
            storage: storage,
            limits: { fileSize: 1500 * 1024 },  // 1500 KB
        }).single("file");

        // Middleware to serve static files from the 'uploads' directory
        app.use("/uploads", express.static(uploadDir));

        // Serve the frontend (HTML, CSS, JS)
        app.use(express.static("public"));

        // API to handle file uploads
        app.post("/upload", (req, res) => {
            upload(req, res, (err) => {
                if (err) {
                    return res.status(500).json({ success: false, message: err.message });
                }
                res.status(200).json({ success: true });
            });
        });

        // API to get uploaded files
        app.get("/files", (req, res) => {
            fs.readdir(uploadDir, (err, files) => {
                if (err) {
                    return res.status(500).json({ success: false, message: err.message });
                }
                const fileData = files.map(file => ({
                    name: file,
                    url: `/uploads/${file}`,
                }));
                res.json(fileData);
            });
        });

        app.listen(PORT, () => {
            console.log(`Server running on http://localhost:${PORT}`);
        });
    </script>
</body>
</html>
