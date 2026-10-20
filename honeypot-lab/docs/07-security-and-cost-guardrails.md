# 07 - Security and Cost Guardrails

A honeypot is intentionally exposed. Treat it as hostile from the moment it goes online.

## Security rules

1. **Isolation.** Dedicated resource group, dedicated VNet, no peering, no VPN, no domain join.
2. **No real data or credentials.** Never reuse a password, never sign in to personal or work accounts on the VM, never store keys or tokens on it.
3. **No managed identity with permissions** attached to the honeypot VMs.
4. **Separate admin path.** Administer through Azure Bastion, Just-in-Time access, or a restricted NSG rule from your own IP, not through the bait port.
5. **Monitor for real compromise.** Query 7 in the KQL doc (success after failures). If you see a genuine successful attacker logon, stop the VM and capture evidence rather than just deleting it.
6. **Outbound control.** Consider an NSG outbound rule restricting traffic so a compromised VM cannot be used to attack others (for example, deny outbound except to Azure Monitor and DNS).
7. **Legal.** Only scan or attack-test assets you own. Check your Azure acceptable use policy and local law.
8. **Time-box it.** Delete everything when the lab is done.

## Cost guardrails

| Control | How |
|---|---|
| Budget alert | Cost Management > Budgets, alert at 50%, 80%, 100% |
| Small VM sizes | B-series burstable |
| Log ingestion | Collect only needed event IDs; ingestion is the main Sentinel cost |
| Shutdown | Deallocate VMs when not in use (stops compute charge; disks still billed) |
| Teardown | Delete the resource group at the end |

Check current pricing and any free trial for Sentinel and Log Analytics on Microsoft's pricing pages; terms change.

## Operational checklist

- [ ] Budget alert configured
- [ ] Honeypot subnet has no route to anything valuable
- [ ] No real credentials anywhere in the lab
- [ ] Alert on success-after-failure enabled
- [ ] Teardown date written on resource group tag
