# Linux Server Configuration (Ubuntu) for Amazon AWS

The following instructions detail the configuration of an [Ubuntu](http://www.ubuntu.com/) server (Amazon AWS Ec2 instance) including installation of
applications and securing the firewall. Web application [TheCatalog](https://github.com/thurstonemerson/the-catalog) is installed to an [Apache](https://httpd.apache.org/) web server. 

## Completed Environment

The web application TheCatalog can be accessed via the following URLS:

- http://52.32.98.214/ 
- http://ec2-52-32-98-214.us-west-2.compute.amazonaws.com/

It is possible to ssh into the server as user 'grader':

```
ssh -p 2200 -i ~/.ssh/udacity_key.rsa grader@52.32.98.214
```

## Basic Configuration


1. Clone the project:

    ```
	$ git clone https://github.com/thurstonemerson/movie-website.git
	$ cd movie-website
    ```

1. Create and initialize virtualenv for the project:

    ```
	$ mkvirtualenv movie-website
    ```
    
1. Create and initialize virtualenv for the project:

    ```
	$ mkvirtualenv swiss-system-tournament
    ```

