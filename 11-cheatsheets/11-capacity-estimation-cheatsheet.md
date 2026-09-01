# Capacity Estimation & Hardware Limits Cheat Sheet

## ⏱️ Universal Constants
- $1\text{ Day} = 86,400\text{ seconds} \approx \mathbf{10^5\text{ seconds}}$
- $1\text{ Million requests/day} \approx \mathbf{11.5\text{ QPS}}$
- $100\text{ Million requests/day} \approx \mathbf{1,160\text{ QPS}}$
- $1\text{ Billion requests/day} \approx \mathbf{11,570\text{ QPS}}$

---

## ⚡ Latency Numbers Every Engineer Must Know
- **L1 CPU Cache**: $0.5\text{ ns}$
- **RAM Access**: $100\text{ ns}$
- **NVMe SSD Read**: $100\text{ }\mu\text{s}$
- **Datacenter Round Trip**: $0.5\text{ ms}$
- **Cross-Country Round Trip (US East to West)**: $60\text{ ms}$
- **Transatlantic Round Trip (US to Europe)**: $150\text{ ms}$
