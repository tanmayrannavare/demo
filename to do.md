✅ Recommended Final Steps:
Build the image:
```
docker build -t student-apache .
```
Tag for Docker Hub:
```
docker tag student-apache yourdockerhubusername/student-apache:latest
```
Login & Push:
```
docker login
docker push yourdockerhubusername/student-apache:latest
```
