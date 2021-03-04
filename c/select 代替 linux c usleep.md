```c
#include <sys/select.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
 
void local_sleep (long sec, long usec)
{
 
  struct timeval timeout = {sec, usec};
  int ret = 0;
 
  if ((0 == timeout.tv_sec) || (timeout.tv_usec < 20))
  {
    printf("local sleep error! input sleep time must greater than 20ms !\n");
    timeout.tv_usec = 20;
  }
 
  ret = select(0, NULL, NULL, NULL, &timeout);
 
  if ((-1 == ret) || (ret))
  {
    printf("local sleep error!\n");
  }

}
```
