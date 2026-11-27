docker build -t mynginx .


Run:
docker run -d -p 8081:80 --name mycontainer mynginx


Verify:
docker ps


Access:
http://localhost:8081


Stop:
docker stop mycontainer


Remove:
docker rm mycontainer


This single sample project is enough for the common Nginx single-container experiment.
