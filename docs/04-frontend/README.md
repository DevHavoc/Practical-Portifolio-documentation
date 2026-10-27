# Frontend Case Studies

[Cloud Case Studies](../../README.md)

## Documentation

- [User Access Overview UI](15-user-access-overview-ui.md) · [code example](15-user-access-overview-ui.md#code-example)
- [Handling POST Requests That Return Binary Data](18-requestmodel-post-blob.md) · [code example](18-requestmodel-post-blob.md#code-example)
- [User Access Queries and N+1 Investigation](19-user-access-batch-query.md) · [code example](19-user-access-batch-query.md#code-example)
- [List Filtering, Sorting and Bulk Action Consistency](29-list-filtering-sorting-and-bulk-actions.md) · [code example](29-list-filtering-sorting-and-bulk-actions.md#code-example)
- [Summary Cards, Result Counts and Pagination](33-summary-cards-result-counts-and-pagination.md) · [code example](33-summary-cards-result-counts-and-pagination.md#code-example)
- [Keeping Old Search Responses Out of the Current Screen](41-stale-search-responses-and-sync-status.md) · [code example](41-stale-search-responses-and-sync-status.md#code-example)
- [Fixing Broken Icons in a GitHub Profile README](45-github-profile-svg-links-and-image-proxies.md) · [code example](45-github-profile-svg-links-and-image-proxies.md#code-example)

## Code example

```javascript
return CostOpsService.getDre(
    this._dreDataInicio,
    this._dreDataFim,
    sFiscalizadores,
    abort.signal,
    function (sync) {
        if (generation !== that._dreGeneration || abort.signal.aborted) return;
        that._setDreSyncStatus(sync);
    }
)
    .then(function (oData) {
        if (generation !== that._dreGeneration || abort.signal.aborted) return;
        var aList = oData.data || oData || [];
        oModel.setProperty(
            "/dreConsolidadoList",
            that._buildDreConsolidadoRows(aList)
        );
        oModel.setProperty(
            "/dreConsolidadoComparativo",
            that._buildDreComparativo(aList)
        );
        oModel.setProperty(
            "/dreConsolidadoCustoAdicional",
            that._buildDreCustoAdicionalComparativo(aList)
        );
        oModel.setProperty(
            "/dreResumo",
            that._buildDreViewModel(
                that._buildDreAgregado(aList)
            )
        );
        oModel.setProperty("/dreConsolidadoLoaded", true);
        oModel.setProperty("/dreLoaded", true);
    })
    .catch(function (error) {
        if (generation !== that._dreGeneration || abort.signal.aborted) return;
        oModel.setProperty("/dreErro", true);
        oModel.setProperty("/dreConsolidadoLoaded", false);
        oModel.setProperty("/dreLoaded", false);

        MessageToast.show(
            (error && error.message) || that.getText("CostOps.Dre.ErroCarregarDre")
        );
    })
    .finally(function () {
        if (oContent && generation === that._dreGeneration) {
            oContent.setBusy(false);
        }
    });
```
