# پایین آوردن حجم داکر ایمیج برای پروژه  Go

در این روش بجای استفاده از ایمیج گو  "golang" برای اجرا برنامه ما ابتدا در داخل ایمیج گو اقدام به بیلد برنامه کردیم و سپس برنامه ی بیلد شده رو به ایمیج آلپاین "alpine" با حجم هفت مگابایت انتقال دادیم,
```ssh

# Start from golang base image
FROM golang:alpine as builder

# Add Maintainer info
LABEL maintainer="mohamad yarahmadi"

# Install git.
# Git is required for fetching the dependencies.
RUN apk update && apk add --no-cache git

# Set the current working directory inside the container 
WORKDIR /app

# Copy go mod and sum files 
COPY go.mod go.sum ./

# Download all dependencies. Dependencies will be cached if the go.mod and the go.sum files are not changed 
RUN go mod download 

# Copy the source from the current directory to the working Directory inside the container 
COPY . .

# Build the Go app
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Start a new stage from scratch
FROM alpine:latest
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy the Pre-built binary file from the previous stage. Observe we also copied the .env file
COPY --from=builder /app/main .      

# Expose port 10000 to the outside world
EXPOSE 10000

#Command to run the executable
CMD ["./main"]
```
در حالت استفاده از ایمیج گو برای اجرای برنامه ایمیج برنامه ما حجمی حدود 975 مگابایت داشت

```sh
samplerestapi   latest    1e1dcde4916d   1 days ago     975MB
```
اما حالا با شرایط جدید ایمیج برنامه ما فقط 13 مگابایت هست
```sh
samplerestapi   latest    dc1e6d1de491   1 days ago     13.1MB
```
