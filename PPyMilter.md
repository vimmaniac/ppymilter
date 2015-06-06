# Pure Python Milter Wiki Page #

PpyMilter is a milter protocol implementation (for sendmail, postfix, etc) that is written entirely in python.  It does not depend on libmilter.a but rather understands the milter protocol and talks directly to your MTA over a (local) network socket.

# PpyMilter Server #

PpyMilter comes with two pre-written network socket servers one asynchronous (great throughput for quick, non blocking milters) and one multi threaded (for more CPU intensive or milters that may block (that themselves depend on disk, network, etc).

# PpyMilter API #

The API is pretty simple.  Take a look at the PpyMilter class in ppymilterbase.py (either the direct code or the pydocs).  The API is a simple python, object-oriented wrapper around the milter protocol.  The API is based on and very similar to the perl pmilter library.

# A Quick Example (To Disallow BCC'ing) #
```
#!/usr/bin/python

import asyncore
from ppymilter import ppymilterbase, ppymilterserver
# (or just "import ppymilterbase, ppymilterserver" depending on your
#  sys.path)

class NoBccMilter(ppymilterbase.PpyMilter):
  def __init__(self):
    ppymilterbase.PpyMilter.__init__(self)
    self.CanAddHeaders()
    self.__mutations = []

  # obviously this doesn't work (well) with mail groups
  def OnRcptTo(self, cmd, rcpt_to, esmtp_info):
    self.__mutations.append(self.AddHeader('To', rcpt_to))
    return self.Continue()

  def OnEndBody(self, cmd):
    tmp = self.__mutations
    self.__mutations = []
    return self.ReturnOnEndBodyActions(tmp)

  def OnResetState(self):
    self.__mutations = []

if __name__ == '__main__':
  ppymilterserver.AsyncPpyMilterServer(port, NoBccMilter)
  asyncore.loop()

```



# FAQ #