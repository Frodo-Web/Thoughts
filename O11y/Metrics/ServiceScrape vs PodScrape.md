# ServiceScrape vs PodScrape
Each pod tracks and exposes it's own metrics. When you create a service monitor it uses the endpoints of the targeted service to dynamically create targets for each pod that is an endpoint of that service. The metrics in prom would then be aggregated on the same job label. <br>
Great explanation. I want to add a little clarifying detail for readers who have less knowledge. When you create a kind: Service in Kubernetes, another resource kind: Endpoint is created for each pod backing the service, and the traffic is routed to those IPs. Those are the endpoints being referred to here.  <br>
Короче, скрейп джобы прометеусов, вмагентов не балансируются через Service при использовании ServiceScrape. Оператор полагается и находит Endpoints тип и прописывает таргеты. Поэтому ServiceScrape скрейпит также все поды. <br>
You actually CAN annotate the Service and scrape a random Pod. But it’s only useful in rare cases. Usually you want to scrape all the Pods, so you annotate the Pod template. <br>
Если где вне куба используются балансеры, то через них ходить не рекомендуется для скрейпинга. <br>
Но например PodScrape можно использовать в случае, когда пода снимается с балансировки (убирается из эндоинтов Service). Например, когда она останавливается или стартует. PodScrape будет собирать метрики. <br>
scraping metric endpoints via a load balancer is an antipattern in prometheus. Best practice is to scrape directly and this is why ServiceMonitor is examining all endpoints in a Service and scraping them this way. <br>
