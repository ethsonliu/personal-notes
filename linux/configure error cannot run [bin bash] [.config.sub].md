configure: error: cannot run /bin/bash ./config.sub

我在目录下并未发现`./config.sub`这个文件，推测可能是`autoreconf -i`没有生成，重新执行一遍`autoreconf -i`出现了该文件，接着`./configure`。

