# Migration Notes

**Migration Date**: 2026-03-30
**Source**: Internal GitLab - AIDLC-DesignReview
**Target**: https://github.com/awslabs/aidlc-workflows
**Target Path**: scripts/aidlc-designreview/

## Migration Status

This directory was migrated from a standalone repository to the aidlc-workflows monorepo.

## What Was Migrated

- ✅ Complete source code (src/design_reviewer/)
- ✅ Full test suite (tests/)
- ✅ Configuration files (config/)
- ✅ Hook installation system (tool-install/)
- ✅ Documentation (README.md, INSTALLATION.md, docs/)
- ✅ Legal files (LICENSE, NOTICE, LEGAL_DISCLAIMER.md)
- ✅ Project configuration (pyproject.toml, uv.lock)

## Post-Migration Checklist

- [ ] Run `uv sync --extra test` to install dependencies
- [ ] Run `pytest` to verify all 743 tests pass
- [ ] Run `design-reviewer --version` to verify CLI works
- [ ] Test hook installation on at least one platform
- [ ] Update aidlc-workflows main README to reference design-reviewer
- [ ] Verify all documentation links work
- [ ] Run security scans on migrated code

## Known Issues

- Hook feature is marked as **EXPERIMENTAL** (see README.md and INSTALLATION.md)

## Original Repository

The original development occurred in the internal GitLab repository.
All Git history and development artifacts remain in that location.

## Contact

For questions about this migration, contact the AIDLC team.
