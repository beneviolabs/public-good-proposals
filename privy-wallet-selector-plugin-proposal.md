# Privy Plugin for NEAR Wallet Selector

## Executive Summary

We propose to develop and open-source `@near-wallet-selector/privy`, a production-ready wallet adapter that integrates Privy's embedded wallets into the NEAR wallet-selector ecosystem. This public good will eliminate significant friction for developers building user-friendly dApps by providing seamless email-based authentication without requiring custom cryptographic implementation on each attempt to use Privy with Near Protocol.

## Problem Statement

### Current Challenges

**Privy's Tier 2 NEAR Support Creates Developer Friction**

While Privy announced NEAR Protocol support in August 2025, it remains a Tier 2 chain (compared to Tier 3 for Ethereum and Solana). This means developers must:

1. **Write Custom Cryptographic Code**: Manually implement transaction signing, message signing (NEP-413), and transaction broadcasting
2. **Create Dual Control Flows**: Maintain separate logic for Privy wallets vs. wallet-selector wallets, introducing conditional branching throughout the application
3. **Spend Significant Time on Boilerplate**: At hackathons and in production, developers lose days building authentication infrastructure instead of core features
4. **Navigate Fragmented Documentation**: Piece together solutions from multiple sources without standardized patterns

**This directly impacts the NEAR ecosystem's ongoing feedback from Developers:**
- **Wallet and Onboarding Gaps**: Users expect email-based login; current solutions are too complex
- **Infrastructure Can Feel Brittle**: Custom implementations lack the robustness of standardized patterns
- **Documentation Fragmented**: No canonical reference for Privy + NEAR integration

### Why This Matters

**User Expectations**: Mainstream users expect email/social login flows. Privy provides this, but integrating it with NEAR is unnecessarily difficult.

**Developer Experience**: The NEAR wallet-selector is the standard pattern for wallet integration. Not having a Privy adapter forces developers to maintain two parallel authentication systems.

**Ecosystem Growth**: Reducing onboarding friction by 10x makes NEAR more competitive for consumer-facing applications where embedded wallets are essential.


## Proposed Solution

### Overview

Create `@near-wallet-selector/privy` - a production-ready, open-source wallet adapter following NEAR's wallet-selector standards ([NEP-408](https://github.com/near/NEPs/blob/master/neps/nep-0408.md), [NEP-368](https://github.com/near/NEPs/pull/368)), enabling developers to integrate Privy with a single function call:

```typescript
const walletSelectorConfig = {
  network: 'mainnet',
  modules: [
    setupPrivyWallet(),  // ← The new hotness
    setupMyNearWallet(),
    setupMeteorWallet(),
  ],
};
```
plus the addition of a client application's registered Privy credentials added into the context

```typescript
// App.tsx
import { PrivyAuthProvider, PrivyWalletBridge } from '@/utils/privy-wallet-selector';

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

The adapter follows the **browser wallet pattern** used by MyNEARWallet:

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
WalletSelectorProvider (NEAR wallet-selector)
    ↓
PrivyWalletBridge (Event handling)
    ↓
Application Code (Standard wallet-selector API)
```

### Security Model

- **Explicit User Consent**: Every signing action requires approval in isolated UI
- **State Isolation**: Privy wallet state is independent from application state
- **Auditable**: Signing context has no access to app state or API keys
- **Standards-Compliant**: Follows NEP-408 (Injected) and NEP-368 (Bridged) wallet standards

### Proof of Concept

We have successfully implemented this solution in production at Peerfolio (closed-source). The implementation:
- ✅ Supports NEP-413 message signing
- ✅ Handles transaction signing and broadcasting
- ✅ Provides seamless email-based authentication
- ✅ Maintains separation of concerns
- ✅ Eliminates conditional wallet logic

**Demo**: Available at request - shows complete flow from email login → transaction signing → callback

## Objectives

### Primary Objectives

1. **Reduce Developer Friction**: Enable Privy + NEAR integration with <10 lines of code
2. **Standardize Best Practices**: Provide canonical reference implementation for embedded wallets on NEAR
3. **Enable Ecosystem Growth**: Lower barrier for consumer-facing dApps to build on NEAR
4. **Create Reusable Public Good**: Open-source, well-documented, maintainable codebase

### Success Criteria

- ✅ Package published to NPM with semantic versioning
- ✅ 3+ production dApps integrate the adapter within 1 month
- ✅ Submission accepted to official near/wallet-selector repository
- ✅ 80%+ test coverage with comprehensive integration tests
- ✅ Documentation rated "clear" by 90%+ of surveyed developers

## Deliverables

### 1. NPM Package: `@near-wallet-selector/privy`

**Features:**
- Complete wallet-selector interface implementation
- NEP-413 message signing support
- Transaction signing and broadcasting
- Network switching (mainnet/testnet)
- TypeScript definitions

**License**: MIT (consistent with wallet-selector ecosystem)

### 2. Comprehensive Documentation

**Developer Docs** (`/docs`):
- Quick start guide (5-minute integration)
- API reference (all methods, types, options)
- Architecture overview (with diagrams)
- Security considerations
- FAQ

### 3. Reference dApp

**Interactive Demo** (`/demo-app`):
- Email-based login/logout
- Message signing (NEP-413)
- Transaction creation and signing
- Network switching
- Error handling demonstrations

**Hosted**: Live demo at `privy-adapter.vercel.app` (or similar)

### 4. Test Suite

**Coverage Target**: >80%

- Unit tests for all core functions
- Integration tests for signing flows
- E2E tests for complete user journeys

### 5. Community Support

- **Office Hours**: 1 sessions per week during the first month post-launch, for developer Q&A
- **Blog Post**: Technical deep-dive explaining architecture and design decisions
- **6 Months Active Maintanence**: Ongoing maintanence of feature requests and issue resolution for the open source repository
- **Video Tutorial**: ~10 minute walkthrough of integration process

## Timeline

**Total Duration**: 6 weeks

**Key Dates**:
- Week 0-2: Project kickoff, near-wallet-selector integration standards complete
- Week 2-3: Open Source Standards complete
- Week 3-4: Testing & Documentation complete
- Week 4: Public release
- Week 5-6: Iteration based on reasonable feedback

## Budget Request

**Total Budget**: $36,250 USD

### Budget Breakdown

| Category | Hours | Rate | Total |
|----------|-------|------|-------|
| **Development** | 180 | $125/hr | $22,500 |
| **Testing & QA** | 50 | $125/hr | $6,250 |
| **Community Support** | 40 | $125/hr | $5,000 |
| **Documentation** | 20 | $125/hr | $2,500 |
| **Total** | 290 | - | **$37,000** |


### Payment Schedule

- **30% upfront** ($10,875): Upon project approval
- **40% at midpoint** ($14,500): Week 3 - Core features complete, tests written
- **30% at completion** ($10,875): Week 6 - Package published, documentation live, PR submitted

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
- ✅ 500+ NPM downloads per month
- ✅ 50+ GitHub stars

**Quality**:
- ✅ 80%+ test coverage
- ✅ Zero critical security issues
- ✅ <24 hour response time to issues

**Documentation**:
- ✅ 90%+ developer satisfaction (post-integration survey)
- ✅ <5 common questions in FAQ (indicates clarity)

### Qualitative Metrics

- **Developer Feedback**: "Made Privy integration 10x easier"
- **Community Recognition**: Recommended by NEARDev
- **Ecosystem Impact**: Hackathon and newly onboarded devs choose Privy w/ Near.

### Sustainability Plan

- **Community Maintenance**: Open contribution model
- **Developer Champions**: Identify and empower power users as maintainers

## References & Prior Art

### NEAR Ecosystem
- [NEAR Wallet Selector Protocol](https://github.com/near/wallet-selector)
- [NEP-408: Injected Wallet Standards](https://github.com/near/NEPs/blob/master/neps/nep-0408.md)
- [NEP-368: Bridged Wallet Standards](https://github.com/near/NEPs/pull/368)
- [Existing Wallet Adapters](https://github.com/near/wallet-selector/tree/main/packages)

### Similar Implementations
- [MyNEARWallet Adapter](https://github.com/near/wallet-selector/blob/main/packages/my-near-wallet) - Reference for browser wallet pattern
- [Privy Swap Example](https://github.com/gagdiez/privy-swap) - Demonstrates Privy + NEAR challenges
- [FastAuth](https://github.com/near/fastauth-wallet) - Similar goals for email-based auth


## Questions for Committee

1. **Coordination**: Is there existing work on embedded wallet adapters we should coordinate with?

2. **Early Adopters**: Which NEAR dApps might be interested in early adoption/testing? Can the committee facilitate introductions?

3. **Privy Partnership**: Is the committee aware of any plans for Privy to build Tier 3 support for NEAR?

4. **Security Review**: What security review process is required for wallet adapters before official inclusion?

5. **Maintenance**: After initial development, is there infrastructure committee support for ongoing maintenance grants?



---

## Appendix: Technical Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                  Application Layer                  │
│  (Your dApp - uses standard wallet-selector APIs)   │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│            NEAR Wallet Selector Core                │
│    (Standard interface for all NEAR wallets)       │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│         @near-wallet-selector/privy                 │
│  • setupPrivyWallet()                               │
│  • PrivyAuthProvider (React Context)                │
│  • PrivyWalletBridge (Event handling)               │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│              Privy SDK (@privy-io/react-auth)       │
│  • User authentication                              │
│  • Embedded wallet creation                         |
|  • Private Key Management                           |
│  • Signing UI components                            |
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│                  NEAR Protocol                      |
│  • Transaction submission                           │
│  • Network validation                               │
└─────────────────────────────────────────────────────┘
```


