By default, Docker containers don't retain any state from _past_ containers. For example, if I:

1. Start a container from an image
    
2. Make some changes to the filesystem (like installing a new package) in that container
    
3. Stop the container
    
4. Start a new container from the same image
    
5. The new container does _not_ have the changes I made in step 2.

However, if I restart the _stopped_ container, it _will_ have the changes I made. This is only worth mentioning because sometimes developers think that killing an old container and starting a new one is the same as _restarting a process_ - but that's not true... it's more like resetting the state of the _entire machine_ to the original image.