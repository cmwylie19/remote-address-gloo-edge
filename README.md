# REMOTE ADDRESS
1. Configure the gateway-proxy service: 
```
k patch svc gateway-proxy -n gloo-system -p '{"spec":{"externalTrafficPolicy":"Local"}}'
```
2. Configure Gateway gateway-proxy:
```
spec:
  bindAddress: '::'
  bindPort: 8080
  httpGateway:
    options:
      httpConnectionManagerSettings:
        useRemoteAddress: true
  options:
    accessLoggingService:
      accessLog:
      - fileSink:
          jsonFormat:
            downstreamRemoteAddress: '%DOWNSTREAM_REMOTE_ADDRESS%'
            duration: '%DURATION%'
            protocol: '%PROTOCOL%'
            upstreamCluster: '%UPSTREAM_CLUSTER%'
            upstreamHost: '%UPSTREAM_HOST%'
            xForwardedFor: '%REQ(X-FORWARDED-FOR)%'
          path: /dev/stdout
  proxyNames:
  - gateway-proxy
  useProxyProto: true
```
3. ```aws elb create-load-balancer-policy --load-balancer-name a405c07f9447e4ebab7ce2c9ea5399b8 --policy-name case-xff-policy --policy-type-name ProxyProtocolPolicyType --policy-attributes AttributeName=ProxyProtocol,AttributeValue=true```
4. ```aws elb set-load-balancer-policies-for-backend-server --load-balancer-name a405c07f9447e4ebab7ce2c9ea5399b8 --instance-port 32455 --policy-names case-xff-policy```
5. ```kubectl annotate svc/gateway-proxy -n gloo-system 'service.beta.kubernetes.io/aws-load-balancer-backend-protocol'=tcp 'service.beta.kubernetes.io/aws-load-balancer-proxy-protocol'='*'```

Search Versions of Helm Chart
```
helm search repo glooe
```