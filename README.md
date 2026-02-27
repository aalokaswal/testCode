function CancelConfirmContinueClick(postbackTarget) {
            if (_cancelConfirmSubmitting) {
                return false;   // already clicked — block completely
            }
            _cancelConfirmSubmitting = true;

            // Disable the Continue button
            var btn = document.getElementById('<%=btnCancelConfirmContinue.ClientID%>');
            if (btn) {
                btn.disabled = true;
                btn.value = 'Please wait...';
            }

            // Show the full-page busy overlay with spinner — stays visible until
            // the browser navigates away to CertificateForm
            var overlay = document.getElementById('divCancelRedirectBusy');
            if (overlay) {
                overlay.style.display = 'block';
            }

            ExecuteButtonClick(postbackTarget);
            return false;
        }
