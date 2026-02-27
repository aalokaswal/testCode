 // Show busy overlay — all styles set here via JS so MasterPage CSS cannot interfere
            var overlay = document.getElementById('divCancelRedirectBusy');
            overlay.style.cssText = [
                'display:block',
                'position:fixed',
                'top:0',
                'left:0',
                'width:100%',
                'height:100%',
                'z-index:99999',
                'background-color:#000000',
                'opacity:0.6',
                'cursor:wait'
            ].join(';');

            var box = document.getElementById('divCancelRedirectBusyBox');
            box.style.cssText = [
                'position:fixed',
                'top:50%',
                'left:50%',
                'margin-top:-50px',
                'margin-left:-90px',
                'width:180px',
                'background:#ffffff',
                'padding:20px',
                'text-align:center',
                'z-index:100000',
                'border:1px solid #cccccc'
            ].join(';');

            var txt = document.getElementById('spnCancelBusyText');
            txt.style.cssText = 'font-size:13px;color:#333333;font-family:Arial,sans-serif;';
