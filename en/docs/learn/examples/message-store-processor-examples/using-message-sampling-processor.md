# How to Use a Message Sampling Processor

This example demonstrates the usage of the message sampling processor.

## Synapse configuration

Following are the artifact configurations that we can use to implement this scenario. See the instructions on how to [build and run](#build-and-run) this example.

=== "Send Sequence"
    ```xml  
    <sequence name="send_seq"  trace="disable"  xmlns="http://ws.apache.org/ns/synapse">
        <call>
            <endpoint key="SimpleStockQuoteServiceEndpoint"/>
        </call>
    </sequence>       
    ```
=== "Proxy Service"    
    ```xml  
    <proxy xmlns="http://ws.apache.org/ns/synapse" name="StockQuoteProxy" transports="https http" startOnLoad="true" trace="disable">
              <description />
        <target>
           <inSequence>
                <log level="full"/>
                <property name="FORCE_SC_ACCEPTED" value="true" scope="axis2"/>
                <property name="OUT_ONLY" value="true"/>
                <store messageStore="MyStore"/>
            </inSequence>
        </target>
    </proxy>
    ```
=== "Message Store"    
    ```xml
    <messageStore name="MyStore" class="org.apache.synapse.message.store.impl.memory.InMemoryStore" xmlns="http://ws.apache.org/ns/synapse" />
    ```
=== "Message Processor"    
    ```xml 
    <messageProcessor xmlns="http://ws.apache.org/ns/synapse"
         class="org.apache.synapse.message.processor.impl.sampler.SamplingProcessor"
         name="SamplingProcessor" messageStore="MyStore">
        <parameter name="interval">20000</parameter>
        <parameter name="sequence">send_seq</parameter>
        <parameter name="is.active">true</parameter>
    </messageProcessor> 
    ```

=== "Endpoint"    
```xml
<endpoint name="SimpleStockQuoteServiceEndpoint" xmlns="http://ws.apache.org/ns/synapse">
    <address uri="http://localhost:9000/services/SimpleStockQuoteService">
        <suspendOnFailure>
            <initialDuration>-1</initialDuration>
            <progressionFactor>1</progressionFactor>
        </suspendOnFailure>
        <markForSuspension>
            <retriesBeforeSuspension>0</retriesBeforeSuspension>
        </markForSuspension>
    </address>
</endpoint>
```

## Build and run

Create the artifacts:

1. {!includes/build-and-run.md!}
2. Create the [proxy service]({{base_path}}/develop/creating-artifacts/creating-a-proxy-service), [endpoint]({{base_path}}/develop/creating-artifacts/creating-endpoints), [mediation sequences]({{base_path}}/develop/creating-artifacts/creating-reusable-sequences), [message store]({{base_path}}/develop/creating-artifacts/creating-a-message-store), and [message processor]({{base_path}}/develop/creating-artifacts/creating-a-message-processor) with the configurations given above.
3. [Deploy the artifacts]({{base_path}}/develop/deploy-artifacts) in your Micro Integrator.

Set up the back-end service:

1. Download the [back-end service](https://github.com/wso2-docs/WSO2_EI/blob/master/Back-End-Service/axis2Server.zip).
2. Extract the downloaded zip file.
3. Open a terminal, navigate to the `axis2Server/bin/` directory inside the extracted folder.
4. Execute the following command to start the axis2server with the SimpleStockQuote back-end service:

    === "On MacOS/Linux"   
          ```bash
          sh axis2server.sh
          ```
    === "On Windows"          
          ```bash
          axis2server.bat
          ```

Send the following request to invoke the service:

```bash
POST http://localhost:8290/services/StockQuoteProxy HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: text/xml;charset=UTF-8
SOAPAction: "urn:getQuote"
Content-Length: 492
Host: localhost:9090
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://services.samples" xmlns:xsd="http://services.samples/xsd">
   <soapenv:Header/>
   <soapenv:Body>
      <ser:getQuote xmlns:ser="http://services.samples" xmlns:xsd="http://services.samples/xsd">
         <ser:request>
            <xsd:symbol>IBM</xsd:symbol>
         </ser:request>
      </ser:getQuote>
   </soapenv:Body>
</soapenv:Envelope>
```

When you send the request, the message will be dispatched to the proxy service. In the proxy service, the store mediator will store the getQuote request message in the "MyStore" message store. The message processor will consume the messages, and forward them to the "send_seq" sequence in configured rate. You will observe that the service invocation rate is not changing when we increase the rate of proxy service invocation.
