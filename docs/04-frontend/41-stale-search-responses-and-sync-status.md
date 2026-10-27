# Keeping Old Search Responses Out of the Current Screen

## Problem

A user can start another search before the first one finishes. If the old response arrives later, it could replace the new result or show the wrong last-update time.

## What changed

I added a request counter and used AbortController to cancel the previous search. Each callback checks the counter and cancellation state before changing the screen. This includes results, errors, update status and the loading indicator.

Cancellation alone is not enough: a response may already be on its way. The counter lets the controller reject that response even then.

## Showing the correct update time

The service only passes the update status to the controller after receiving a complete result. Failed or incomplete streams do not set a new update time. The server sends the status of the saved copy used by that report, so another background update cannot change the meaning of the displayed timestamp.

The screen validates the date and shows it as text. If the report uses the last complete copy while an update is pending, it shows a warning and keeps the original date.

## What was checked

Local frontend tests covered complete results, failed and incomplete streams, invalid status data and callbacks from cancelled searches. This was a code check, not a production browser review. The update-status feature belonged to the background work that had not been integrated or deployed during this session.

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
