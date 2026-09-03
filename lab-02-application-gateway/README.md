# Lab 02 — Application Gateway

## Task
**Two backends, one entrance**

Complete the Microsoft Quickstart to create an Application Gateway (L7 load balancer) using the Azure CLI and distribute traffic across two backend web servers.

## Reference
https://learn.microsoft.com/en-us/azure/application-gateway/quick-create-cli

## Architecture

```
VNet: myVNet (10.21.0.0/16) — Germany West Central
│
├── myAGSubnet     (10.21.0.0/24)   ← Application Gateway only (dedicated subnet)
│   └── myAppGateway                 ← Standard_v2, capacity 2, public frontend
│       ├── Frontend IP:  myAGPublicIPAddress (public, Standard)
│       ├── Frontend port: 80
│       ├── Listener:     appGatewayHttpListener
│       ├── Backend pool: appGatewayBackendPool → 10.21.1.4, 10.21.1.5
│       ├── HTTP setting: appGatewayBackendHttpSettings (HTTP :80)
│       └── Rule:         rule1 (Basic)
│
└── myBackendSubnet (10.21.1.0/24)  ← backend VMs + NAT Gateway
    ├── myVM1 (10.21.1.4) — nginx, serves "Hello World from host myVM1!"
    ├── myVM2 (10.21.1.5) — nginx, serves "Hello World from host myVM2!"
    ├── myNSG  → AllowHTTPFromAppGw: TCP 80 from 10.21.0.0/24
    └── myNatGateway + myNatPip     ← outbound internet for VMs
```

## Resources Deployed (12 core)

| Resource | Name | Notes |
|---|---|---|
| Virtual Network | myVNet | 10.21.0.0/16 |
| Subnet | myAGSubnet | 10.21.0.0/24, Application Gateway only |
| Subnet | myBackendSubnet | 10.21.1.0/24, backend VMs |
| Public IP | myAGPublicIPAddress | Standard, App Gateway frontend |
| Public IP | myNatPip | Standard, NAT Gateway |
| NAT Gateway | myNatGateway | Outbound internet for backend subnet |
| NSG | myNSG | AllowHTTPFromAppGw (TCP 80 from 10.21.0.0/24) |
| NIC | myNic1, myNic2 | Attached to VM1/VM2 |
| VM (backend) | myVM1 | Ubuntu, nginx installed via cloud-init |
| VM (backend) | myVM2 | Ubuntu, nginx installed via cloud-init |
| Application Gateway | myAppGateway | Standard_v2, capacity 2, Generation_1 |

All resources in resource group **myResourceGroupAG** (Germany West Central).

## Key Configuration Notes

- **SKU**: Standard_v2 (autoscale-capable v2 tier), fixed capacity 2 for the lab
- **App Gateway needs its own subnet** — `myAGSubnet` holds only `myAppGateway`; backend VMs live in `myBackendSubnet`
- **NSG rule** allows HTTP only from the App Gateway subnet (10.21.0.0/24), not the internet, so the VMs cannot be reached directly
- **NAT Gateway** on the backend subnet gives the VMs deterministic outbound IPs for package installs
- Each VM serves a page identifying itself, so alternating responses prove load balancing

## Test Result

```
$ az network application-gateway show-backend-health -g myResourceGroupAG -n myAppGateway
10.21.1.4  Healthy   Success. Received 200 status code
10.21.1.5  Healthy   Success. Received 200 status code

$ for i in 1..8; do curl -s http://<AG-public-ip>/; done
Hello World from host myVM2!
Hello World from host myVM2!
Hello World from host myVM1!
Hello World from host myVM2!
Hello World from host myVM1!
Hello World from host myVM2!
Hello World from host myVM1!
Hello World from host myVM2!
```

Both backends healthy; App Gateway distributes requests across `myVM1` and `myVM2`. ✅

## Artifacts

Exported from the deployed resource group for reproducibility:

- `lab02-artifacts/appgw.json` — full `az network application-gateway show` output
- `lab02-artifacts/template.json` — exported ARM template of the resource group
- `lab02-artifacts/resources.txt` — `az resource list` for `myResourceGroupAG`
- `lab02-artifacts/backend-health.txt` — App Gateway backend-health check
- `lab02-artifacts/curl-alternating.txt` — 8 sequential curls showing alternating backends
- `lab02-artifacts/nsg-rules.txt` — NSG inbound rules

## Redeploy Notes

This lab was built with the **Azure CLI** (not an ARM template deployment). To redeploy, follow the CLI quickstart linked above; substitute your own admin password and public SSH key. `lab02-artifacts/template.json` is the exported template of the deployed resource group and can be used as a reference, but was not the deployment mechanism.

## Screenshots

- `screenshots/RG.png` — resource group overview
- `screenshots/myResourceGroupAG.png` — App Gateway topology
- `screenshots/vm1.png`, `screenshots/vm2.png` — nginx page from each backend
