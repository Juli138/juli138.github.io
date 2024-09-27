const http = require('http');
const fs = require('fs');
const path = require('path');

const readFileAsync = (filePath) => {
    return new Promise((resolve, reject) => {
        fs.readFile(filePath, 'utf8', (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve(data);
            }
        });
    });
};

const server = http.createServer(async (req, res) => {
    if (req.method === 'GET') {
        if (req.url === '/') {
            try {
                const data = await readFileAsync(path.join(__dirname, 'index.html'));
                res.writeHead(200, { 'Content-Type': 'text/html' });
                res.end(data);
            } catch (err) {
                console.error(err);
                res.writeHead(500, { 'Content-Type': 'text/html' });
                res.end('<h1>500 Internal Server Error</h1>');
            }
        } else {
            try {
                const data = await readFileAsync(path.join(__dirname, '404.html'));
                res.writeHead(404, { 'Content-Type': 'text/html' });
                res.end(data);
            } catch (err) {
                console.error(err);
                res.writeHead(500, { 'Content-Type': 'text/html' });
                res.end('<h1>500 Internal Server Error</h1>');
            }
        }
    } else {
        res.writeHead(405, { 'Content-Type': 'text/html' });
        res.end('<h1>405 Method Not Allowed</h1>');
    }
});

const PORT = 8080;
server.listen(PORT, () => {
    console.log(Server is running on http://localhost:${PORT});
});
