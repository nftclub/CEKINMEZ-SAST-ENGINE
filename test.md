CEKINMEZ SAST: Complex IoC & Cross-File Vulnerability Analysis
This document demonstrates the CEKINMEZ SAST ENGINE's ability to track tainted data through complex Inversion of Control (IoC) patterns and multi-layered dependency injections—a scenario where most standard SAST tools and LLMs fail due to context fragmentation.

🎯 The Challenge: "The Final Boss" Scenario
This test case involves a distributed data flow across 5 files. The objective is to detect a Business Logic Leak and NoSQL Injection hidden behind a dependency injection container and fragmented object destructuring.

1. The Entry Point (Source)
contracts.ts receives the raw user input.

TypeScript
// File: contracts.ts
import { container } from '../container/ioc';

export const patchContract = async (req: any, res: any) => {
    const result = await container.contractService.patchContract(
        req.params.contractId,
        req.user.orgId,
        req.user.id,
        req.body  // ← UNTRUSTED SOURCE
    );
    res.json(result);
};
2. The IoC Bridge
zor.js acts as the dependency injection hub, a common "blind spot" for legacy static analysis.

JavaScript
// File: zor.js
import { ContractRepository } from '../repositories/ContractRepository';
import { AuditRepository } from '../repositories/AuditRepository';
import { ContractService } from '../services/ContractService';

export const container = {
    contractRepo: new ContractRepository(),
    auditRepo: new AuditRepository(),
    get contractService() {
        return new ContractService(this.contractRepo, this.auditRepo);
    }
}; //
3. The Logic Layer (Fragmentation)
ContractService.ts sanitizes some fields but leaks the raw object to the audit trail.

TypeScript
// File: ContractService.ts
export class ContractService {
    async patchContract(contractId: string, orgId: string, userId: string, body: Record<string, unknown>) {
        const contract = await this.contractRepo.findByIdAndOrg(contractId, orgId);
        if (!contract) throw new Error('Not found');

        const { title, value } = body; // Partially validated
        await this.contractRepo.patch(contractId, { title, value });

        await this.auditRepo.write({
            userId,
            contractId,
            payload: body  // ← RAW UNTRUSTED BODY LEAKED
        });
        return contract;
    }
}
4. The Sink Points
The untrusted data reaches the database execution layer.

Audit Repository (NoSQL Sink):

TypeScript
// File: AuditRepository.ts
export class AuditRepository {
    async write(entry: Record<string, unknown>) {
        return db.audit.insertOne(entry); // ← CRITICAL VULNERABILITY
    }
}
Contract Repository (Update Sink):

TypeScript
// File: ContractRepository.ts
export class ContractRepository {
    async patch(id: string, data: Record<string, unknown>) {
        return db.contracts.update(data, { where: { id } }); //
    }
}
🚀 The Result: CEKINMEZ Master Report
CEKINMEZ successfully pierced the IoC abstraction, pre-loading the entire architecture into its Unified Memory Pool to identify the exact intersection of the logic breakdown.

JSON
[
  {
    "id": 1,
    "file": "/targets/AuditRepository.ts",
    "line": 6,
    "risk": "CRITICAL",
    "rule": "WAF_BLIND_AST_IDOR",
    "message": "[AST-IDOR] Titanium Pierced! Destructured user input reaches 'insertOne' sink point through fragmented logic.",
    "cwe": "CWE-943 / CWE-915"
  },
  {
    "id": 2,
    "file": "/targets/ContractRepository.ts",
    "line": 9,
    "risk": "CRITICAL",
    "rule": "WAF_BLIND_AST_IDOR",
    "message": "[AST-IDOR] Titanium Pierced! Destructured user input reaches 'update' sink point.",
    "cwe": "CWE-915"
  }
]
🦈 Execution Summary
Deterministic Flow: The engine tracked the req.body source through 4 procedural hops and an IoC container.

Zero Noise: Unlike legacy tools that flag every database call, CEKINMEZ only flagged the specific path where unvalidated object keys reach a write operation.

Architecture Integrity: Proved that fragmented business logic cannot hide vulnerabilities from a high-fidelity AST analysis.
