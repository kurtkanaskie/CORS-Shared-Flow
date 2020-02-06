# CORS-Shared-Flow
Single Shared Flow that works in both Flow Hooks or within Proxy.

The Shared Flow  `cors-v1` uses conditions to determine where it is being executed and does the right thing:
* Captures information during the OPTIONS preflight.
* Sets headers in the response.

## Use in Flow Hooks

Attach Shared Flow to both Flow Hooks:
* Pre Proxy
* Post Proxy

When attaching to the Shared Flow that will be used in the Flow Hooks (e.g. pre-proxy and post-proxy) the `currentstep.flowname` flow variable must be past as a parameter, otherwise its empty.
```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<FlowCallout async="false" continueOnError="false" enabled="true" name="FC-CORS">
    <DisplayName>FC-CORS</DisplayName>
    <Parameters>
        <!-- Need to set currentstep.flowstate, its empty otherwise -->
        <Parameter name="currentstep.flowstate">{currentstep.flowstate}</Parameter>
    </Parameters>
    <SharedFlowBundle>cors-v1</SharedFlowBundle>
</FlowCallout>
```

## Use in Proxy directly

Alternatively it can be added to the proxy directly:
* ProxyPreFlow or OPTIONS conditional flow
* ProxyPost flow and DefaultFaultRule

Attach the Shared Flow either in ProxyPreFlow or a conditional flow name `CORS` for the `OPTIONS` verb.
Attach the Shared Flow both in the ProxyPostFlow and the DefaultFaultRule.

Here's a simple template for a proxy that works with the Shared Flow used in Flow Hooks.
Note the use of the `OPTIONS` conditional flow and the no-route conditional RouteRules.

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">
    <PreFlow name="PreFlow">
        <Request>
            <Step>
                <Condition>(request.verb != "OPTIONS")</Condition>
                <Name>VA-header</Name>
            </Step>
            <Step>
                <Condition>(request.verb != "OPTIONS")</Condition>
                <Name>AM-remove-x-apikey</Name>
            </Step>
        </Request>
        <Response/>
    </PreFlow>
    <PostFlow name="PostFlow">
        <Request/>
        <Response/>
    </PostFlow>
    <Flows>
        <Flow name="ping">
            <Description>proxy health check</Description>
            <Request/>
            <Response>
                <Step>
                    <Name>AM-create-ping-response</Name>
                </Step>
            </Response>
            <Condition>(proxy.pathsuffix MatchesPath "/ping") and (request.verb = "GET")</Condition>
        </Flow>
        <Flow name="status">
            <Description>back end health check</Description>
            <Request/>
            <Response>
                <Step>
                    <Name>AM-create-status-response</Name>
                </Step>
            </Response>
            <Condition>(proxy.pathsuffix MatchesPath "/status") and (request.verb = "GET")</Condition>
        </Flow>
        <Flow name="CORS">
            <Description>ignore OPTIONS request</Description>
            <Request/>
            <Response/>
            <Condition>(request.verb = "OPTIONS") and (proxy.pathsuffix MatchesPath "/**")</Condition>
        </Flow>
        <Flow name="catch all">
            <Description>Catch any unmatched calls and raise fault</Description>
            <Request>
                <Step>
                    <Name>RF-path-suffix-not-found</Name>
                </Step>
            </Request>
            <Response/>
            <Condition>(proxy.pathsuffix MatchesPath "**")</Condition>
        </Flow>
    </Flows>
    <HTTPProxyConnection>
        <BasePath>/pingstatus/v1</BasePath>
        <VirtualHost>secure</VirtualHost>
    </HTTPProxyConnection>
    <RouteRule name="cors">
        <Condition>(request.verb == "OPTIONS")</Condition>
    </RouteRule>
    <RouteRule name="ping">
        <Condition>(proxy.pathsuffix MatchesPath "/ping")</Condition>
    </RouteRule>
    <RouteRule name="default">
        <TargetEndpoint>default</TargetEndpoint>
    </RouteRule>
</ProxyEndpoint>
```

## Disclaimer

This example is not an official Google product, nor is it part of an official Google product.

## License

This material is copyright 2019, Google LLC. and is licensed under the Apache 2.0 license.
See the [LICENSE](LICENSE) file.
