# Offline

We've already done a bit of networking in Docker. Specifically, we've exposed containers to the host network on various ports and accessed web traffic.

Now, let's force a container into _offline_ mode!

You might be thinking, "why would I want to turn off networking?!?" Well, usually it's for security reasons. You might want to remove the network connection from a container in one of these scenarios:

- You're running 3rd party code that you don't trust, and it shouldn't need network access
    
- You're building an e-learning site, and you're allowing students to execute code on your machines
    
- You know a container has a virus that's sending malicious requests over the internet, and you want to do an audit
    

## Network None

The `docker run` command has a `--network none` flag that makes it so that the container can't network with the outside world, which is super useful for isolating containers.