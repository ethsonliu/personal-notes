Static variables with the local scope are created, when they are used the first time. 

This lazy creation is a guarantee that C++98 provides. With C++11, static variables with the local scope are also initialized in a thread-safe way.

