# Changing the Default Ports with Offset

When you run multiple runtimes on the same server or virtual machines (VMs), you must change their default ports with an `offset` value to avoid port conflicts. An offset defines the number by which all ports in the runtime (e.g., HTTP/S ports) are increased. 

For example, if the default HTTP port is 9763 and the offset is 1, the effective HTTP port changes to 9764. For each additional WSO2 product instance running on the same machine, you set the port offset to a unique value.

There are two ways to set an offset to a port: Update the server configurations, or pass the port offset during server startup. See the instructions given below to port offset.

## Before you begin

See the complete list of [default ports]({{base_path}}/install-and-setup/setup/reference/default-product-ports) in Micro Integrator.

Note that most of the **runtime ports** change automatically based on the offset you specify here.

## Changing the default MI ports

The default port offset in the WSO2 Micro Integrator runtime is `10`. Use one of the following two methods to apply an offset to the Micro Integrator runtime.

!!! Tip
	-	The internal offset of 10 is overriden by this manual offset. That is, if the manual offset is 3, the default ports will change as follows:
		- `8290` -> `8283` (8290 - 10 + 3)
		- `8253` -> `8246` (8253 - 10 + 3)
		- `9164` -> `9157` (9164 - 10 + 3)
	-	Note that if you manually set an offset of 10 using the following method, you will get the same default ports.

#### Update the server configurations

1. Stop the MI server if it is already running.

2.  Open the `<MI_HOME>/conf/deployment.toml` file.

3.  Uncomment the `offset` parameter under `[server]` and set the offset value.
    
    === "Format"
        ```toml 
        [server]
        offset=<offset_value>
        ```
    === "Example"        
        ```toml 
        [server]
        offset = 3
        ```

4. [Restart the server]({{base_path}}/install-and-setup/install/running-the-mi).

#### Pass the port offset during server startup

1. Stop the MI server if it is already running.

2. Restart the server with the `-DportOffset` system property.

    - Linux/Mac OS
   
        === "Format"         
            ```toml 
            ./micro-integrator.sh -DportOffset=<offset_value>
            ```
        === "Example"            
            ```toml 
            ./micro-integrator.sh -DportOffset=3
            ```
        
    - Windows
   
        === "Format"        
            ```toml 
            micro-integrator.bat -DportOffset=<offset_value>
            ```
        === "Example"            
            ```toml 
            micro-integrator.bat -DportOffset=3
            ```

## What's Next?

You need to restart the server for these changes to take effect.