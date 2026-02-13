# Heading

---
slug: untitled-page-95oa
kb: default-kb
tier: 2
status: draft
updated_at: 2026-02-13T20:54:53.724Z
---

qweqwe qweqwe

qewqwe qwe eqw qwe eqw

# Heading

asdasdwqewqewqe ???

> `useEffect(() => { if (!selectedSnippet) { setSnippetTitleDraft(''); setSnippetSlugDraft(''); setSnippetTierDraft(2); setSnippetEditorMd(''); setSnippetTags(''); setSnippetOwnerTeam('Core'); setSnippetLastSavedHash(''); setSnippetLastSavedAt(null); setSnippetAutosaveState('saved'); setSnippetAutosaveError(null); return; } const nextTags = (selectedSnippet.tags ?? []).join(', '); const nextTier = selectedSnippet.tier === 1 ? 1 : 2; setSnippetTitleDraft(selectedSnippet.title); setSnippetSlugDraft(selectedSnippet.slug); setSnippetTierDraft(nextTier); setSnippetEditorMd(selectedSnippet.contentMd); setSnippetTags(nextTags); setSnippetOwnerTeam(selectedSnippet.ownerTeam || 'Core'); setSnippetDetailTab('edit'); setHistoryDiffMode('previous'); setSnippetUsageView('tree'); setSnippetUsageFilterKbId(null); setSnippetHighlightedUsage(null); const nextSavedHash = buildSnippetDraftHash({ title: selectedSnippet.title, slug: selectedSnippet.slug, contentMd: selectedSnippet.contentMd, tier: nextTier, status: selectedSnippet.status, tags: nextTags, ownerTeam: selectedSnippet.ownerTeam || 'Core' }); setSnippetLastSavedHash(nextSavedHash); setSnippetLastSavedAt(selectedSnippet.updatedAt ? new Date(selectedSnippet.updatedAt).toISOString() : new Date().toISOString()); setSnippetAutosaveState('saved'); setSnippetAutosaveError(null); }, [selectedSnippet]);`

asd
