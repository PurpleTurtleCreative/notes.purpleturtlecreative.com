---
title: Connecting Containers
parent: Docker
nav_order: 1
---

# How to Let Separate Docker Containers Communicate

If you're trying to make requests between Docker containers, it's not as straight-forward as updating the URLs for the requests.

Here's how to let Docker containers communicate and resolve connection errors such as:
```
cURL error 7: Failed to connect to localhost port 8080: Connection refused
```

## Add the Docker containers to the same network

The Docker containers must be in the same network to communicate.

```bash
$ docker network create my_network
$ docker network connect my_network myapi_php_1
$ docker network connect my_network mysite_wordpress_1
```

You can then confirm the containers are connected to the same network by executing:
```bash
$ docker network inspect my_network
```

Confirm by looking in the returned JSON at the paths `Containers.[id].Name` to confirm the container names are listed.

## Make requests by using the container's Name

Now you may be surprised that you still can't get the containers to communicate.

What you need to do is use the container's name in place of the domain name for your HTTP request:

| Wrong | Correct |
| --- | --- |
| `http://localhost:8080/api/v2/resource/123` | `http://myapi_php_1/api/v2/resource/123` |

## Connecting automatically

If you need the containers to always be in the same network, look at adding the `networks` key to your `docker-compose.yml` file.