Static variables with the local scope are created, when they are used the first time. 

This lazy creation is a guarantee that C++98 provides. With C++11, static variables with the local scope are also initialized in a thread-safe way.

而且只初始化一次，见 https://stackoverflow.com/questions/5567529/what-makes-a-static-variable-initialize-only-once
