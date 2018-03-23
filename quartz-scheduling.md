# Quartz

### Issues with Quartz Instance ID with Cluster

`SimpleInstanceIdGenerator` tries to generate instance ID using `InetAddress.getLocalHost().getHostName()` that can sometimes fail, when Java is not able to resolve DNS name of the machine. `InetAddress.getHostName()` is doing a reverse lookup on the server's IP address using the naming service \(DNS\) configured in your O/S.

```
import org.apache.commons.lang3.StringUtils;
import org.quartz.spi.InstanceIdGenerator;

import java.util.Random;

/**
 * Generates ID based on system HOSTNAME. If HOSTNAME is not available, it generates random ID.
 */
public class HostnameInstanceIdGenerator implements InstanceIdGenerator {

    public static final String HOSTNAME_KEY = "HOSTNAME";

    @Override
    public String generateInstanceId() {
        String id = System.getenv(HOSTNAME_KEY);
        if (StringUtils.isBlank(id)) {
            // get random ID if hostname is not available
            id = String.valueOf(new Random().nextLong());
        }
        return id;
    }
}
```

Then we need to set our new ID generator in Quartz properties. 

```
org.quartz.scheduler.instanceIdGenerator.class=mypackage.HostnameInstanceIdGenerator
```



