# [AWS] Configure DNS Firewall Allowlist (Console)

Created: 2025-12-09
Tags: \#aws \#dns-firewall \#route53 \#security \#cli

## English Summary

Configuration notes for AWS Route 53 Resolver DNS Firewall.
To implement a "whitelist" approach (Allow specific domains, Block everything else), two rules are required within a Rule Group: an ALLOW rule with higher priority and a BLOCK rule (wildcard `*`) with lower priority.

## Japanese Summary

AWS Route 53 Resolver DNS Firewall で、特定のドメインのみ許可し、それ以外をすべて遮断する（ホワイトリスト方式）設定の備忘録。
マネジメントコンソールとCLI両方の手順を記載。ポイントは「許可リスト」を優先度高く、「全拒否リスト（`*`）」を優先度低く設定すること。

-----

## Configuration / Code

### 1\. Strategy (設定方針)

DNS Firewall does not have an implicit "Deny All". You must explicitly create a block rule.
DNS Firewallにはデフォルトの「全拒否」設定がないため、明示的に拒否ルールを作成する必要があります。

  * **Priority 100:** ALLOW List (Target domains)
  * **Priority 200:** BLOCK List (Wildcard `*`)

### 2\. Management Console Steps (マネコン手順)

#### Step 1: Create Domain Lists (ドメインリスト作成)

Go to **VPC \> DNS Firewall \> Domain lists**.
Create two lists:

**List A: `allowed-domains`**
Add the following domains:

```text
*.amazonaws.com
*.aws.amazon.com
*.awsapps.com
*.okta.com
*.oktacdn.com
*.cloudfront.net
```

**List B: `blocked-all`**
Add the following domain:

```text
*
```

#### Step 2: Create Rule Group (ルールグループ作成)

Go to **VPC \> DNS Firewall \> Rule groups**.
Create `rg-strict-allowlist`.

#### Step 3: Add Rules (ルール追加)

Inside the Rule Group, add rules in this order:

| Priority | Name | Domain List | Action |
| :--- | :--- | :--- | :--- |
| **100** | `rule-allow-targets` | `allowed-domains` (List A) | **ALLOW** |
| **200** | `rule-block-others` | `blocked-all` (List B) | **BLOCK** |

#### Step 4: Associate VPC (VPCに関連付け)

Associate the Rule Group with the target VPC.

-----

