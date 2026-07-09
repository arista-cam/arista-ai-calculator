# AI Backend Network Calculator

A browser-based tool for sizing AI backend network fabrics. Enter your GPU cluster parameters and get switch counts, optics requirements, cabling breakdowns, and topology diagrams — all based on real Arista 800G switch platforms.

## Features

- **Four fabric tiers** — single-tier (collapsed spine), two-tier (leaf-spine Clos), three-tier (super-spine / spine / leaf), and DES (Distributed Etherlink Switch)
- **Platform suggestions** — suggests the smallest balanced switch combination based on cluster size, with Apply/Dismiss controls
- **Full optics breakdown** — per-device transceiver counts (spine, leaf uplink/downlink, host NIC) with Conventional/LPO toggle (LPO available at 800G only), OSFP form factor for all switch ports
- **Interactive topology** — D3-powered diagrams with multi-plane view; click any compute node to expand GPU/NIC detail with plane-colored connectivity lines
- **Multi-plane NIC breakout** — correctly models single-NIC hosts across multiple planes (e.g. 1×800G NIC → 4×200G links, one per plane)
- **Spine-link breakout** — host port breakout (e.g. 800G → 2×400G) automatically applies to spine-facing uplinks too, doubling effective connectivity with 800G breakout optics
- **Built-in validation** — hard errors for invalid topologies and warnings for unbalanced fabrics, with minimum platform suggestions
- **Light/dark theme** — follows system preference automatically
- **Print-friendly** — clean output for reports and proposals

## Supported Arista Platforms

### Standard Leaf-Spine (Tiers 1–3)

Traditional Clos topologies using independent switches interconnected with standard Ethernet links.

| Platform | Ports | Type |
|----------|-------|------|
| 7060X6 | 32 / 64 | Fixed 800G |
| 7800R4 | 144 / 288 / 432 / 576 | Chassis (4/8/12/16-slot × 36-port line cards) |

Use standard platforms when you need flexible topology design, custom oversubscription ratios, or clusters that don't fit the DES constraints.

### DES — Distributed Etherlink Switch (Tier 4)

The 7700R4 DES is a cell-based, VOQ-scheduled fabric that presents as a single non-blocking logical switch. It uses fixed-function leaf and spine modules rather than independent switches.

| Component | SKU | Ports |
|-----------|-----|-------|
| Leaf | DCS-DL-7700R4 | 18 Ethernet OSFP (host-facing) + 20 fabric OSFP |
| Spine | DCS-DS-7720R4 | 128 fabric-only OSFP |

**Key constraints:**
- One-stage max: 128 leaves, 40 spines per pod
- Host breakout: 1×800G or 2×400G per Ethernet port (no 4-way breakout)
- Always non-blocking (1:1 subscription) — no oversubscription tuning
- Max ~4,608 host ports per plane (128 leaves × 18 ports × 2 breakout)

**When to use DES:** Best for mid-size AI clusters (up to ~4,600 GPUs per plane) that benefit from a simplified, non-blocking fabric without traditional leaf-spine topology planning. The cell-based architecture eliminates oversubscription decisions and provides deterministic latency.

## Usage

Open `index.html` in any modern browser. No build step, no dependencies, no server required.

1. Enter your GPU count (or node count + GPUs per node)
2. Select a fabric tier
3. Adjust switch configuration (planes, downlink:uplink ratio, breakout)
4. Click **Calculate Network Requirements**

The tool suggests optimal switch platforms based on your inputs. Apply the suggestion or choose manually via the dropdowns. Select "Other (custom)" for arbitrary port counts. When DES is selected, switch configuration is automatic — leaf/spine selectors are hidden.

## Configuration Options

| Input | Description |
|-------|-------------|
| Total GPUs / Nodes | Cluster size — syncs bidirectionally |
| GPUs per node | Typically 8 for modern GPU servers |
| Links per GPU | Backend NIC links (1 standard, 2 for dual-rail) |
| Fabric tier | Single, two-tier, three-tier Clos, or DES |
| Leaf / Spine switch | Arista platform or custom port count |
| Planes | Redundant parallel fabrics (1–8) |
| Downlink:Uplink ratio | Port allocation ratio (1 = non-blocking, <1 = more uplinks) |
| Host port breakout | Switch port splitting (1:1, 1:2, 1:4) — applies to both host-facing and spine-facing ports |
| Spine / Host link speed | Spine link speed auto-syncs with breakout (e.g. 1:2 → 400G). 100G–800G host links |

## Output

- **Metrics grid** — total GPUs, backend links, spine/leaf/total switch counts
- **Switch breakdown** — per-tier tables with platform names, switch counts per plane, port utilization, and bundle sizes
- **Interactive topology diagram** — D3-powered visualization with multi-plane view, click-to-expand GPU/NIC detail showing per-GPU connectivity to leaf switches with plane-colored links
- **Optics breakdown** — transceiver counts by tier and type (QSFP28, QSFP-DD, OSFP) with grand total, plus a **Conventional / LPO** toggle (shown only when 800G optics are in the design) to switch between standard DSP-based and Linear Pluggable Optics
- **Estimated power consumption** — optics and switch power with a **Typical / 80% / Max** toggle. Switch power figures sourced from Arista platform datasheets (excludes optics, calculated separately). LPO mode shows total power savings vs conventional
