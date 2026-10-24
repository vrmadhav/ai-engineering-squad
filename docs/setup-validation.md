# Squad setup validation

Date: October 23, 2026 (America/Los_Angeles)
Scope: reusable squad configuration, instructions, and templates only.
Base commit: `d46b3b38aa22463dd82399d977b55a895a76becb`; bootstrap changes are uncommitted.
Environment: macOS arm64, Python 3.9.6, Codex CLI `0.162.0-alpha.17.2`.

## Result

The configuration/documentation setup passed static checks and independent QA.
Native Codex diagnostics loaded configuration, and native prompt inspection
included the complete repository `AGENTS.md`. End-to-end selection and execution
of the four custom agent identities remains **NOT VERIFIED**.

No application was built, dependencies added, providers selected, or deployment
performed. The existing MIT license was preserved.

## Evidence

Commands ran from the repository root unless noted.

| Check | Actual outcome |
| --- | --- |
| `codex --version` | Exit 0: `codex-cli 0.162.0-alpha.17.2` |
| `codex --strict-config doctor --summary` | Config check: loaded. Sandbox check: restricted filesystem/network, OnRequest. **Overall exit 1**, due to unreachable desktop update/runtime CDN; this is not an overall passing health check. |
| `codex debug prompt-input 'Validate squad discovery only; do not change files.'` | Exit 0 after an approved execution outside the filesystem sandbox. Output captured in a temporary JSON file; an exact substring check confirmed the entire repository `AGENTS.md` was present. No model task submitted. |
| TOML parse and field assertions using Python's already-installed `pip._vendor.tomli` | Exit 0: all five TOML files parse; all four agent names match filenames; required fields are nonempty strings; only the documented role fields are used; configured agents and sandbox defaults match expected values. |
| UTF-8, final newline, trailing whitespace, NUL, and local Markdown link checks | Passed for all 14 authored/updated files, including this report: 15 local links resolved. All 13 snapshot hashes matched, and the embedded validation snippet exited 0. |
| `git diff --check` | Exit 0. Supplementary file-content checks cover newly created, untracked files too. |
| Independent QA agent `/root/qa_setup_review` | Initial review and final report recheck found no defects. Independently verified all 13 hashes, 14 files, 15 local links, the embedded validation snippet, and `git diff --check`. READY for configuration/documentation delivery; native custom-agent selection NOT VERIFIED. Native prompt-inspection evidence was supplied by the Lead, not independently repeated by QA. |

The initial `codex --strict-config debug prompt-input ...` attempt exited 1
because this version does not support `--strict-config` with `debug`. Retrying
without that flag first hit the filesystem sandbox (exit 1); the approved retry
above succeeded. Strict configuration checking was performed with `doctor`.

The health check also reported WebSocket connectivity, optional MCP environment,
terminal, and local security-assessment warnings. These describe this machine's
environment, not verified defects in the squad files. No global settings or MCP
credentials were changed to suppress them.

## Reproduce static configuration checks

Use Python 3.11+ with standard-library `tomllib`, or the already-installed parser
used here. No repository dependency is required or installed by this snippet.

```sh
python3 - <<'PY'
from pathlib import Path
try:
    import tomllib
except ModuleNotFoundError:
    from pip._vendor import tomli as tomllib

config = tomllib.loads(Path('.codex/config.toml').read_text())
assert config['agents'] == {
    'enabled': True,
    'max_concurrent_threads_per_session': 3,
}
assert config['approval_policy'] == 'on-request'
assert config['sandbox_mode'] == 'workspace-write'
expected = {'architect', 'frontend', 'backend', 'qa'}
paths = list(Path('.codex/agents').glob('*.toml'))
assert {p.stem for p in paths} == expected
for path in paths:
    agent = tomllib.loads(path.read_text())
    assert set(agent) == {'name', 'description', 'developer_instructions'}
    assert agent['name'] == path.stem
    assert all(isinstance(v, str) and v.strip() for v in agent.values())
print('PASS: project config and four agent definitions')
PY
```

If neither parser is installed, use an available Python 3.11+ interpreter rather
than adding an application dependency merely for this check. This validates TOML
and this squad's documented field subset, not the complete Codex runtime schema.

## Reviewed content snapshot

SHA-256 values identify the bootstrap content reviewed before this evidence
document was added. The report itself is excluded to avoid a self-referential hash.

| File | SHA-256 |
| --- | --- |
| `AGENTS.md` | `bd97b554999bcf72486bd34d02a83d014f9a1bd416986f1c5cfc68d626d40890` |
| `README.md` | `c2abb10079c5bf86d4d44b9e56897c4db6ee673b403f0400292a2e75f9eced84` |
| `.codex/config.toml` | `a6c20cdb5a5edb7cf69fbce4c0b36067122d584c6ea330d6172999f2c179df35` |
| `.codex/agents/architect.toml` | `ca5f3a22e7b3d487ca76f4ba274298da9a0585fd8c303f062ab9558ee7b69b59` |
| `.codex/agents/frontend.toml` | `b3cd38ae5f757353b944259ccc44791a7c73a8d5b6b799d1d801dba76c28f0f6` |
| `.codex/agents/backend.toml` | `a00ddaea3de1a3a530ce58c5ceb4878c438e63f24e6c07920d701e59f199701d` |
| `.codex/agents/qa.toml` | `5c770ef46f26a46b2fecb69c441438dda7201d3fe877f57a84dba3ccbb5637d2` |
| `templates/product-requirements.md` | `8ce481fb72a4de35597eb8371bf1587083997cd08a569939245fec267df7a9bb` |
| `templates/architecture-decision.md` | `bf7c3afac40799ac67221486a19fd9a5888484092c7294cc02972d29925c95b6` |
| `templates/api-contract.md` | `397b584c5a6281e6edd26fbd5c374bc7fd0eaaac7df0b0600e4c3f2fe2c54b40` |
| `templates/implementation-plan.md` | `97dbe4612781c7bd1603cad38383761b6624209dbfa37e4d3bebbab689058428` |
| `templates/qa-report.md` | `b352b8cbdda8616997ca96802504949d6a3bfa85f3a13c0213ab9e8f271f439b` |
| `templates/release-readiness.md` | `f831d72faf81ae4f40a00142e7e6e9d5495f0c18f2955cdac3f1653ba2a65fda` |

## Verification limits and next check

- The independent QA run was a real separate subagent instructed to read the QA
  role file. This session's delegation interface does not expose a custom-agent
  identity selector, so that run does not prove automatic selection of `qa`.
- Static syntax, a successful config-load diagnostic, and AGENTS.md discovery do
  not establish that every custom agent has been selected and executed. Run the
  fresh-session smoke prompt in [README.md](../README.md) to verify all four.
- Application lint, type checks, tests, build, CI, security, performance, and
  production readiness are N/A for this configuration-only bootstrap. They must
  be established and executed when a real product is created.
- Behavioral rules are not technical deployment locks. Configure production
  permission boundaries and protected delivery environments when a real pipeline
  exists; no pipeline or provider has been chosen here.

The source format and discovery guidance were verified against the fetched
[official subagent documentation](https://developers.openai.com/codex/multi-agent),
[configuration basics](https://learn.chatgpt.com/docs/config-file/config-basic),
and [AGENTS.md discovery](https://learn.chatgpt.com/docs/agent-configuration/agents-md).
