
var _cancelConfirmSubmitting = false;   // lives in page memory, never reset

function CancelConfirmContinueClick(postbackTarget) {
    if (_cancelConfirmSubmitting) {
        return false;    // second click → blocked immediately, nothing fires
    }
    _cancelConfirmSubmitting = true;     // first click → flag set permanently
    // ... disable button visually + trigger postback
}
-----------------------

VB — update Page_Load to call CancelConfirmContinueClick instead of ExecuteButtonClick directly:
