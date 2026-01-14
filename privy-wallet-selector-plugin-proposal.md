# Privy Embedded Wallet Adapter for near-connect

## Executive Summary

We propose to develop and open-source a production-ready Privy embedded wallet adapter for **near-connect** (formerly hot-connect), enabling seamless email-based authentication for NEAR dApps. This public good will eliminate significant friction for developers building user-friendly dApps by providing standardized Privy integration without requiring custom cryptographic implementation on each attempt to use Privy with Near
Protocol.


## Problem Statement

### Current Challenges

**Privy's Tier 2 NEAR Support Creates Developer Friction**

While Privy announced NEAR Protocol support in August 2025, it remains a Tier 2 chain (compared to Tier 3 for Ethereum and Solana). This means developers must:

1. **Write Custom Cryptographic Code**: Manually implement transaction signing, message signing (NEP-413), and transaction broadcasting
2. **Create Dual Control Flows**: Maintain separate logic for Privy wallets vs. standard wallet connections, introducing conditional branching throughout the application
3. **Spend Significant Time on Boilerplate**: At hackathons and in production, developers lose days building authentication infrastructure instead of core features
4. **Navigate Fragmented Documentation**: Piece together solutions from multiple sources without standardized patterns

**This directly impacts the NEAR ecosystem's ongoing feedback from Developers:**
- **Wallet and Onboarding Gaps**: Users expect email-based login; current solutions are too complex
- **Infrastructure Can Feel Brittle**: Custom implementations lack the robustness of standardized patterns
- **Documentation Fragmented**: No canonical reference for Privy + NEAR integration

### Why This Matters

**User Expectations**: Mainstream users expect email/social login flows. Privy provides this, but integrating it with NEAR is unnecessarily difficult.

**Developer Experience**: The near-connect framework is emerging as a streamlined pattern for wallet integration. Not having a Privy adapter forces developers to maintain custom authentication systems if they want to leverage Privy's recent support of Near Protocol.

**Ecosystem Growth**: Reducing onboarding friction by 10x makes NEAR more competitive for consumer-facing applications where embedded wallets are essential.


## Proposed Solution

### Overview

Create a production-ready, open-source Privy wallet adapter for [near-connect](https://github.com/azbang/near-connect), enabling developers to integrate Privy with minimal configuration:

```typescript

// App.tsx

import { NearConnector } from '@hot-labs/near-connect';

import '@peerfolio/privy-near-adapter';

const connector = new NearConnector({ network: 'mainnet' });

```

plus the addition of a client application's registered Privy credentials added into the context

```typescript

// App.tsx

import { PrivyAuthProvider, PrivyWalletBridge } from '@peerfolio/privy-near-adapter';

function App() {
  return (
    <PrivyProvider appId="..." clientId="...">
      <PrivyAuthProvider>
        <WalletSelectorProvider config={walletSelectorConfig}>
          <PrivyWalletBridge />
          <YourApp />
        </WalletSelectorProvider>
      </PrivyAuthProvider>
    </PrivyProvider>
  );
}
```


### Architecture

The adapter follows the **injected browser wallet pattern**:

1. **Sign Request**: Application calls `wallet.signMessage()` or `wallet.signAndSendTransaction()`
2. **Modal Navigation**: User navigates to dedicated signing route with request details
3. **User Approval**: Isolated signing context for explicit consent
4. **Callback**: Redirect back to the developer's custom UI with signed results

**Key Components:**

```
PrivyProvider (Privy SDK)
    ↓
PrivyAuthProvider (Unified React Context)
    ↓
NearConnectProvider (near-connect core)
    ↓
PrivyWalletBridge (Event handling)
    ↓
Application Code (Standard near-connect API)
```

### Security Model

- **Explicit User Consent**: Every signing action requires approval in isolated UI
- **State Isolation**: Privy wallet state is independent from application state
- **Auditable**: Signing context has no access to app state or API keys
- **Standards-Compliant**: Follows NEP-408 (Injected) and NEP-368 (Bridged) wallet standards

### Proof of Concept

We have successfully implemented this solution in production at Peerfolio (closed-source) integrated with near-wallet-selector. The implementation:
- ✅ Supports NEP-413 message signing
- ✅ Handles transaction signing and broadcasting
- ✅ Provides seamless email-based authentication
- ✅ Maintains separation of concerns
- ✅ Eliminates conditional wallet logic

**Demo**: Available at request - shows complete flow from email login → transaction signing → callback

**Note** Privy's SDK requires local storage for session persistence, so near-connect's sandbox pattern likely wouldn't be feasible without support from Privy. However, with near-connect supporting injected wallets running within the application's context, we will adapt the above prototype to validate that this approach within near-connect's architecture. Detailed in the milestones below.

## Objectives

### Primary Objectives

1. **Reduce Developer Friction**: Enable Privy + NEAR integration with minimal configuration
2. **Standardize Best Practices**: Provide canonical reference implementation for embedded wallets on NEAR via near-connect
3. **Enable Ecosystem Growth**: Lower barrier for consumer-facing dApps to build on NEAR
4. **Create Reusable Public Good**: Open-source, well-documented, maintainable codebase

### Success Criteria

- ✅ Package published to NPM with semantic versioning
- ✅ 3+ production dApps integrate the adapter within 3 months
- ✅ Adapter compatible with near-connect (or fallback to wallet-selector if required)
- ✅ 80%+ test coverage with comprehensive integration tests
- ✅ Documentation rated "clear" by 90%+ of surveyed developers

## Deliverables

### 1. NPM Package: Privy Adapter for near-connect

**Features:**
- Complete near-connect adapter interface implementation
- NEP-413 message signing support
- Transaction signing and broadcasting
- Network switching (mainnet/testnet)
- TypeScript definitions

**License**: MIT


### 2. Comprehensive Documentation
Submit PR to hot-connect repository adding `privy-wallet` to repository/manifest.json:
```
{
  "id": "privy-wallet",
  "name": "Privy",
  "description": "Email & social login with embedded NEAR wallet",
  "icon": "https://...",
  "type": "injected",
  "features": {
    "signMessage": true,
    "signAndSendTransaction": true,
    "signAndSendTransactions": true,
    "signInWithoutAddKey": true
  }
}
```

### 3. Comprehensive Documentation

**Developer Docs** (`/docs`):
- Quick start guide for wallet-selector integration
- Quick start guide for hot-connect integration
- API reference (all methods, types, options)
- Architecture overview (with diagrams)
- Security considerations
- FAQ

### 4. Reference dApp

**Interactive Demo** (`/demo-app`):
- Email-based login/logout
- Message signing (NEP-413)
- Transaction creation and signing
- Network switching
- Error handling demonstrations

**Hosted**: Live demo at a publicly accessible URL

### 5. Test Suite

**Coverage Target**: >80%

- Unit tests for all core functions
- Integration tests for signing flows
- E2E tests for complete user journeys

### 5. Community Support (Launch Phase)

- **Office Hours**: 1 session per week during the first month post-launch for developer Q&A
- **Blog Post**: Technical deep-dive explaining architecture and design decisions
- **Video Tutorial**: ~10 minute walkthrough of integration process

## Maintenance & Support (12 Months)

This maintenance period is tightly scoped to ensure sustainable, high-quality support.

### Included in Maintenance

- **Critical Bug Fixes**: Issues that prevent core functionality from working
- **Security Patches**: Addressing vulnerabilities in the adapter code
- **Dependency Compatibility Updates**: Maintaining compatibility with NEAR, Privy SDK, and near-connect updates
- **Issue Triage and Response**: Reviewing, categorizing, and responding to reported issues

### Excluded from Maintenance

The following are explicitly outside the scope of this maintenance commitment:

- New feature development
- Performance optimizations beyond critical fixes
- Major refactors or architectural changes
- Support for additional wallet standards or networks
- Custom integrations or consulting

### Capacity and Response Expectations

- **Monthly Capacity Cap**: ~8 hours per month
- **Work Beyond Cap**: Any work exceeding this capacity requires a separate written agreement or change order
- **Issue Acknowledgment**: Within 3 business days
- **Critical Issue Resolution**: Best-effort resolution within 5 business days

## Compatibility / Fallback Clause

**Primary Target**: near-connect

This proposal targets near-connect as the primary integration framework based on current ecosystem guidance. However, near-connect is a newer framework and we have not yet validated a production prototype against it.

**Validation Commitment**: The Validation Phase (Week 1) will confirm that Privy's signing flows work end-to-end with near-connect.

**Fallback Provision**: If, during validation or subsequent development, we identify blocking incompatibilities that cannot be resolved in collaboration with the near-connect team within 3 months, we may fall back to implementing the adapter for the NEAR wallet-selector (using the Injected wallet standard per NEP-408).

## Timeline

**Total Duration**: 7 weeks

| Week | Phase | Activities |
|------|-------|------------|
| 1 | **Validation Phase** | Confirm Privy signing flows (message + transaction) work with near-connect. Deliverable: Validated prototype or documented blockers. |
| 2-3 | Core Development | Implement near-connect adapter, authentication bridge, and core signing flows |
| 4 | Integration Standards | Achieve 80%+ test coverage, complete developer documentation, satisfy PR expectations |
| 5 | Quality |  Testing & Documentation complete |
| 6 | Public Release | NPM publication, demo app deployment, blog post |
| 7 | Iteration | Address reasonable feedback, refinements based on early adopter input |

**Key Milestones**:
- Week 1: Validation complete (go/no-go for near-connect)
- Week 3: Core features complete
- Week 5: Testing & Documentation complete
- Week 6: Public release
- Week 7: Initial feedback incorporated

## Budget Request

**Total Budget**: $45,000 USD

### Budget Rationale

This budget reflects:
- The addition of a dedicated **Validation Phase** to de-risk near-connect integration
- An extended **12-month maintenance window** with defined scope and capacity

### Budget Breakdown

| Category | Hours | Rate | Total |
|----------|-------|------|-------|
| **Core Development** | 180 | $125/hr | $22,500 |
| **Validation Phase** | 40 | $125/hr | $5,000 |
| **Testing & QA** | 50 | $125/hr | $6,250 |
| **Documentation** | 20 | $125/hr | $2,500 |
| **Community Support (Launch)** | 20 | $125/hr | $2,500 |
| **12-Month Maintenance** | 50 | $125/hr | $6,250 |
| **Total** | **360** | - | **$45,000** |


### Payment Schedule

- **30% upfront** ($13,500): Upon project approval
- **40% at midpoint** ($18,000): Week 4 - Core features complete, validation confirmed, tests written
- **30% at completion** ($13,500): Week 7 - Package published, documentation live, 12-month maintenance period begins

## Team & Qualifications

### Core Team

**Charles (@openwebeconomy)** - Developer, Project Lead
- Founder at Peerfolio, House of Stake Delegate, long-time Near contributor and advocate.
- Experience integrating Privy + NEAR in production environments
- Deep understanding of NEAR and wallet-selector
- Active NEAR community contributor

**Ryan Marvin** - Developer
- 10 yrs as a software engineer and architect in SaaS and consumer tech.
- Experience integrating Privy + NEAR in production environments
- Deep experience in python, LLMs, langchain, and near-wallet-selector patterns

**Jacob Nall** - Designer
- 12 yrs designing products across automotive, fintech, and blockchain.
- Active NEAR community contributor.


## Success Metrics

### Quantitative Metrics

**Adoption** (3 months post-launch):
- ✅ 3+ production dApps using the adapter
- ✅ 10+ NPM downloads per month
- ✅ 5+ GitHub stars

**Quality**:
- ✅ 80%+ test coverage
- ✅ Zero critical security issues
- ✅ Issue acknowledgment within 3 business days

**Documentation**:
- ✅ 90%+ developer satisfaction (post-integration survey)
- ✅ <5 common questions in FAQ (indicates clarity)

### Qualitative Metrics

- **Developer Feedback**: "Made Privy integration 10x easier"
- **Community Recognition**: Recommended by NEAR developer community
- **Ecosystem Impact**: Hackathon and newly onboarded devs choose Privy with NEAR

### Sustainability Plan

- **Community Maintenance**: Open contribution model
- **Developer Champions**: Identify and empower power users as maintainers

## References & Prior Art

### near-connect Ecosystem
- [near-connect (hot-connect)](https://github.com/AhaLabs/near-connect) - Primary integration target
- near-connect adapter patterns (to be validated during Validation Phase)

### NEAR Ecosystem
- [NEAR Wallet Selector Protocol](https://github.com/near/wallet-selector) - Fallback integration target
- [NEP-408: Injected Wallet Standards](https://github.com/near/NEPs/blob/master/neps/nep-0408.md)
- [NEP-368: Bridged Wallet Standards](https://github.com/near/NEPs/pull/368)
- [Existing Wallet Adapters](https://github.com/near/wallet-selector/tree/main/packages)
- [near connect](https://github.com/azbang/near-connect)

### Similar Implementations
- [MyNEARWallet Adapter](https://github.com/near/wallet-selector/blob/main/packages/my-near-wallet) - Reference for browser wallet pattern
- [Privy Swap Example](https://github.com/gagdiez/privy-swap) - Demonstrates Privy + NEAR challenges
- [FastAuth](https://github.com/near/fastauth-wallet) - Similar goals for email-based auth


## Answered Questions
1. Is there existing work on embedded wallet adapters we should coordinate with? We should design to support [hot-connect](https://github.com/azbang/near-connect), rather than wallet selector.
2. Which NEAR dApps might be interested in early adoption/testing?  PingPay, Peerfolio, and it is our responsibility to find others from EcoCollab and directly contacting the Hot Wallet team.
3. Privy Partnership: Is the committee aware of any plans for Privy to build Tier 3 support for NEAR? No.
4. **Security Review**: What security review process is required for near-connect adapters before official inclusion? It depends on PR review from the hot-connect team.


