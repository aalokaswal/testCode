

USE [BEFS_ADMIN]
GO
/****** Object:  StoredProcedure [dbo].[Calcions]    Script Date: 06-02-2026 08:19:46 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[Calcions]
    @DSTAMPOF DATETIME,
    @ALLFLAG CHAR(1)
AS
BEGIN
    SET NOCOUNT ON;

    -- Variable Declarations
    DECLARE
        @SAT100 NUMERIC(12,4),
        @PSAT NUMERIC(12,4),
        @RVAL VARCHAR(1),
        @LOOPER INT,
        @DATEOFX VARCHAR(20),
        @XFLUORIDE NUMERIC(14,6),
        @INCVAL2 INT = 5000,
        @TOUT INT = 5000,
        @INCVAL INT = 1,
        @CNTER INT = 0,
        @TDS_CALC NUMERIC(14,6),
        @TDS_RAT NUMERIC(14,6),
        @CATIONS NUMERIC(14,6),
        @BICARB NUMERIC(14,6),
        @CARB NUMERIC(14,6),
        @ANIONS NUMERIC(14,6),
        @CAAN_RAT NUMERIC(14,6),
        @TDS_SP NUMERIC(14,6),
        @TDS_CALSP NUMERIC(14,6),
        @ORTHOTOT NUMERIC(14,6),
        @NITROCHK NUMERIC(14,6),
        @TSS_TURB NUMERIC(14,6),
        @NACL_RAT NUMERIC(14,6),
        @SPSO4_RAT NUMERIC(14,6),
        @CAT_SP NUMERIC(14,6),
        @AN_SP NUMERIC(14,6),
        @TDS_CALCR VARCHAR(1),
        @TDS_RATR VARCHAR(1),
        @CATIONSR VARCHAR(1),
        @BICARBR VARCHAR(1),
        @CARBR VARCHAR(1),
        @ANIONSR VARCHAR(1),
        @CAAN_RATR VARCHAR(1),
        @TDS_SPR VARCHAR(1),
        @TDS_CALSPR VARCHAR(1),
        @ORTHOTOTR VARCHAR(1),
        @NITROCHKR VARCHAR(1),
        @TSS_TURBR VARCHAR(1),
        @NACL_RATR VARCHAR(1),
        @SPSO4_RATR VARCHAR(1),
        @CAT_SPR VARCHAR(1),
        @AN_SPR VARCHAR(1),
        @PPERVAL FLOAT,
        @PMAXVAL FLOAT,
        @PSTD FLOAT;

    -- MESSAGE BUILDING VARIABLE - WIDENED TO AVOID OVERFLOW
    DECLARE @MSG_TEXT VARCHAR(4000);

    -- Declare variables for the cursor columns
    -- Replace with actual data types and sizes
    DECLARE
        @PROG_CODE VARCHAR(50),
        @SITE_NAME VARCHAR(50),
        @COL_DATEX VARCHAR(20),
        @COL_DATE DATE,
        @COL_TIME VARCHAR(10),
        @DEPTH NUMERIC(14,6),
        @LASMPLID VARCHAR(50),
        @ALKALINTY NUMERIC(14,6),
        @CALCIUM NUMERIC(14,6),
        @CHLORIDE NUMERIC(14,6),
        @MAGNESIUM NUMERIC(14,6),
        @NITRATE NUMERIC(14,6),
        @NO2_NO3 NUMERIC(14,6),
        @POTTASIUM NUMERIC(14,6),
        @SILICA NUMERIC(14,6),
        @SODIUM NUMERIC(14,6),
        @SULFATE NUMERIC(14,6),
        @PHOSPHU NUMERIC(14,6),
        @TOTHARD NUMERIC(14,6),
        @FLUORIDE NUMERIC(14,6),
        @PHFIELD NUMERIC(14,6),
        @PHLAB NUMERIC(14,6),
        @MANGANESE NUMERIC(14,6),
        @ORTH_PHOS NUMERIC(14,6),
        @ORTH_PHOSR VARCHAR(1),
        @KJELDAHL NUMERIC(14,6),
        @KJELDAHLR VARCHAR(1),
        @AMMONIA NUMERIC(14,6),
        @AMMONIAR VARCHAR(1),
        @TSS NUMERIC(14,6),
        @TURBIDITY NUMERIC(14,6),
        @DISOXY NUMERIC(14,6),
        @TEMP_CENT NUMERIC(14,6),
        @SPEC_COND NUMERIC(14,6),
        @DSTAMP DATETIME,
        @LABNUM VARCHAR(50);

    -- Declare the cursor
    DECLARE data_cursor CURSOR FORWARD_ONLY READ_ONLY FOR
    SELECT
        PROG_CODE, SITE_NAME, COL_DATEX, COL_DATE, COL_TIME, DEPTH, LASMPLID,
        ALKALINTY, CALCIUM, CHLORIDE, MAGNESIUM, NITRATE, NO2_NO3, POTTASIUM,
        SILICA, SODIUM, SULFATE, PHOSPHU, TOTHARD, FLUORIDE, PHFIELD, PHLAB,
        MANGANESE, ORTH_PHOS, ORTH_PHOSR, KJELDAHL, KJELDAHLR, AMMONIA, AMMONIAR,
        TSS, TURBIDITY, DISOXY, TEMP_CENT, SPEC_COND, DSTAMP, LABNUM
    FROM dbo.SC_CHM_ALL
    WHERE (@ALLFLAG = 'Y') OR (DSTAMP = @DSTAMPOF)
    ORDER BY SITE_NAME, COL_DATEX, COL_TIME;

    -- Initialize DATEOFX
    IF @ALLFLAG = 'Y'
    BEGIN
        DELETE FROM dbo.SCIONS;
        SET @DATEOFX = CONVERT(VARCHAR(20), GETDATE(), 120);
    END
    ELSE
    BEGIN
        SET @DATEOFX = CONVERT(VARCHAR(20), @DSTAMPOF, 120);
    END

    OPEN data_cursor;

    FETCH NEXT FROM data_cursor INTO
        @PROG_CODE, @SITE_NAME, @COL_DATEX, @COL_DATE, @COL_TIME, @DEPTH, @LASMPLID,
        @ALKALINTY, @CALCIUM, @CHLORIDE, @MAGNESIUM, @NITRATE, @NO2_NO3, @POTTASIUM,
        @SILICA, @SODIUM, @SULFATE, @PHOSPHU, @TOTHARD, @FLUORIDE, @PHFIELD, @PHLAB,
        @MANGANESE, @ORTH_PHOS, @ORTH_PHOSR, @KJELDAHL, @KJELDAHLR, @AMMONIA, @AMMONIAR,
        @TSS, @TURBIDITY, @DISOXY, @TEMP_CENT, @SPEC_COND, @DSTAMP, @LABNUM;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        SET @CNTER += 1;

        IF @CNTER > 120000
            BREAK;

        -- Delete existing records if ALLFLAG is not 'Y'
        IF @ALLFLAG <> 'Y'
        BEGIN
            DELETE FROM dbo.SCIONS
            WHERE SITE_NAME = @SITE_NAME AND COL_DATEX = @COL_DATEX AND COL_TIME = @COL_TIME;
        END

        -- Skip records where PHFIELD > 11
        IF @PHFIELD IS NOT NULL AND @PHFIELD > 11
        BEGIN
            FETCH NEXT FROM data_cursor INTO
                @PROG_CODE, @SITE_NAME, @COL_DATEX, @COL_DATE, @COL_TIME, @DEPTH, @LASMPLID,
                @ALKALINTY, @CALCIUM, @CHLORIDE, @MAGNESIUM, @NITRATE, @NO2_NO3, @POTTASIUM,
                @SILICA, @SODIUM, @SULFATE, @PHOSPHU, @TOTHARD, @FLUORIDE, @PHFIELD, @PHLAB,
                @MANGANESE, @ORTH_PHOS, @ORTH_PHOSR, @KJELDAHL, @KJELDAHLR, @AMMONIA, @AMMONIAR,
                @TSS, @TURBIDITY, @DISOXY, @TEMP_CENT, @SPEC_COND, @DSTAMP, @LABNUM;
            CONTINUE;
        END

        -- Initialize variables
        SET @TDS_CALC = NULL; SET @TDS_RAT = NULL; SET @CATIONS = NULL; SET @BICARB = NULL;
        SET @CARB = NULL; SET @ANIONS = NULL; SET @CAAN_RAT = NULL; SET @TDS_SP = NULL;
        SET @TDS_CALSP = NULL; SET @ORTHOTOT = NULL; SET @NITROCHK = NULL; SET @TSS_TURB = NULL;
        SET @NACL_RAT = NULL; SET @SPSO4_RAT = NULL; SET @CAT_SP = NULL; SET @AN_SP = NULL;

        SET @TDS_CALCR = NULL; SET @TDS_RATR = NULL; SET @CATIONSR = NULL; SET @BICARBR = NULL;
        SET @CARBR = NULL; SET @ANIONSR = NULL; SET @CAAN_RATR = NULL; SET @TDS_SPR = NULL;
        SET @TDS_CALSPR = NULL; SET @ORTHOTOTR = NULL; SET @NITROCHKR = NULL; SET @TSS_TURBR = NULL;
        SET @NACL_RATR = NULL; SET @SPSO4_RATR = NULL; SET @CAT_SPR = NULL; SET @AN_SPR = NULL;

        -- Perform Calculations (Example for TDS_CALC)
        IF @ALKALINTY IS NOT NULL AND @CALCIUM IS NOT NULL AND
           @CHLORIDE IS NOT NULL AND @MAGNESIUM IS NOT NULL AND
           (@NITRATE IS NOT NULL OR @NO2_NO3 IS NOT NULL) AND
           @POTTASIUM IS NOT NULL AND @SILICA IS NOT NULL AND
           @SODIUM IS NOT NULL AND @SULFATE IS NOT NULL AND
           @PHOSPHU IS NOT NULL AND @TOTHARD IS NOT NULL AND
           @FLUORIDE IS NOT NULL
        BEGIN
            IF @NITRATE IS NOT NULL
            BEGIN
                SET @TDS_CALC = (@ALKALINTY * 0.6) + @CALCIUM + @CHLORIDE +
                                @MAGNESIUM + (@NITRATE * 4.45) + @POTTASIUM +
                                @SILICA + @SODIUM + @SULFATE + @FLUORIDE +
                                (@PHOSPHU * 3.06);
                SET @TDS_CALCR = ' ';
            END
            ELSE
            BEGIN
                SET @TDS_CALC = (@ALKALINTY * 0.6) + @CALCIUM + @CHLORIDE +
                                @MAGNESIUM + (@NO2_NO3 * 4.45) + @POTTASIUM +
                                @SILICA + @SODIUM + @SULFATE + @FLUORIDE +
                                (@PHOSPHU * 3.06);
                SET @TDS_CALCR = ' ';
            END
        END

        -- Continue with other calculations following the same pattern...

        -- Call ANISTATS procedure if necessary
        IF @SPSO4_RAT IS NOT NULL AND @ALLFLAG <> 'Y'
        BEGIN
            EXEC dbo.ANISTATS1
                'ION',
                @SITE_NAME,
                'SPSO4_RAT',
                0.95,
                @PPERVAL OUTPUT,
                @PMAXVAL OUTPUT,
                @PSTD OUTPUT;

            IF ROUND(@SPSO4_RAT, 4) > ROUND(@PMAXVAL, 4) AND @PMAXVAL > 0
            BEGIN
                -- BUILD MESSAGE STRING INCREMENTALLY TO AVOID NVARCHAR OVERFLOW
                SET @MSG_TEXT  = 'D:';
                SET @MSG_TEXT  = @MSG_TEXT + SUBSTRING(@COL_DATEX, 5, 2) + '/';
                SET @MSG_TEXT  = @MSG_TEXT + SUBSTRING(@COL_DATEX, 7, 2) + '/';
                SET @MSG_TEXT  = @MSG_TEXT + SUBSTRING(@COL_DATEX, 1, 4);
                SET @MSG_TEXT  = @MSG_TEXT + ' T:' + @COL_TIME;
                SET @MSG_TEXT  = @MSG_TEXT + ' SPSO4_RAT: OLD MAX:';
                SET @MSG_TEXT  = @MSG_TEXT + CAST(ROUND(@PMAXVAL, 3) AS VARCHAR(20));
                SET @MSG_TEXT  = @MSG_TEXT + ' Sample Value: ';
                SET @MSG_TEXT  = @MSG_TEXT + LTRIM(CAST(@SPSO4_RAT AS VARCHAR(20)));

                INSERT INTO dbo.SCUPRPT 
                VALUES (
                    @DATEOFX,
                    @INCVAL,
                    @MSG_TEXT,
                    'Z',
                    @SITE_NAME
                );
                SET @INCVAL += 1;
            END
            -- Continue with other conditions...
        END

        -- Insert the calculated values into SCIONS
        INSERT INTO dbo.SCIONS
        (
            PROG_CODE, SITE_NAME, COL_DATEX, COL_DATE, COL_TIME, COL_DEPTH,
            LASMPLID,
            TDS_CALC, TDS_CALCR, TDS_RAT, TDS_RATR, CATIONS, CATIONSR, BICARB, BICARBR,
            CARB, CARBR, ANIONS, ANIONSR, CAAN_RAT, CAAN_RATR, TDS_SP, TDS_SPR,
            TDS_CALSP, TDS_CALSPR, ORTHOTOT, ORTHOTOTR, NITROCHK, NITROCHKR,
            TSS_TURB, TSS_TURBR, NACL_RAT, NACL_RATR, SPSO4_RAT, SPSO4_RATR,
            CAT_SP, CAT_SPR, AN_SP, AN_SPR, PRCNT_SAT, PRCNT_SATR, DSTAMP, LABNUM, DATEOF
        )
        VALUES
        (
            @PROG_CODE, @SITE_NAME, @COL_DATEX, @COL_DATE, @COL_TIME, @DEPTH, @LASMPLID,
            @TDS_CALC, @TDS_CALCR, @TDS_RAT, @TDS_RATR, @CATIONS, @CATIONSR, @BICARB, @BICARBR,
            @CARB, @CARBR, @ANIONS, @ANIONSR, @CAAN_RAT, @CAAN_RATR, @TDS_SP, @TDS_SPR,
            @TDS_CALSP, @TDS_CALSPR, @ORTHOTOT, @ORTHOTOTR, @NITROCHK, @NITROCHKR,
            @TSS_TURB, @TSS_TURBR, @NACL_RAT, @NACL_RATR, @SPSO4_RAT, @SPSO4_RATR,
            @CAT_SP, @CAT_SPR, @AN_SP, @AN_SPR, @PSAT, @RVAL, @DSTAMP, @LABNUM, @DATEOFX
        );

        -- Fetch the next record
        FETCH NEXT FROM data_cursor INTO
            @PROG_CODE, @SITE_NAME, @COL_DATEX, @COL_DATE, @COL_TIME, @DEPTH, @LASMPLID,
            @ALKALINTY, @CALCIUM, @CHLORIDE, @MAGNESIUM, @NITRATE, @NO2_NO3, @POTTASIUM,
            @SILICA, @SODIUM, @SULFATE, @PHOSPHU, @TOTHARD, @FLUORIDE, @PHFIELD, @PHLAB,
            @MANGANESE, @ORTH_PHOS, @ORTH_PHOSR, @KJELDAHL, @KJELDAHLR, @AMMONIA, @AMMONIAR,
            @TSS, @TURBIDITY, @DISOXY, @TEMP_CENT, @SPEC_COND, @DSTAMP, @LABNUM;
    END

    CLOSE data_cursor;
    DEALLOCATE data_cursor;

END






----------------------------------------------------------------------------------------------------------------------------------------------------------------------
else if (e.CommandName == "CopyRecord")
{
    string recordId = e.CommandArgument.ToString();

    // Store the original LASMPLID to position the copy after it
    Session["OriginalLasmplid"] = recordId;

    // Copy the record
    string newLasmplid = CopyRecordInDb(recordId);

    if (!string.IsNullOrEmpty(newLasmplid))
    {
        // Store the new ID to highlight it after reload
        Session["NewLasmplid"] = newLasmplid;
        Session["HighlightRowId"] = newLasmplid;

        // Reload grid data with custom ordering
        LoadGridDataWithCopiedRow();

        // Scroll to the newly added row using JavaScript
        string script = @"
            setTimeout(function() {
                var highlightedRow = document.querySelector('tr[style*=""background-color""]');
                if (highlightedRow) {
                    highlightedRow.scrollIntoView({ behavior: 'smooth', block: 'center' });
                }
            }, 100);";
        ScriptManager.RegisterStartupScript(this, GetType(), "ScrollToNew", script, true);
    }
}

  private void LoadGridDataWithCopiedRow()
  {
      string columnName = (Session["ColumnName"] as string) ?? "TEMP_CENT";
      DataTable dt = GetSCChmAll(columnName);

      string originalId = Session["OriginalLasmplid"]?.ToString();
      string newId = Session["NewLasmplid"]?.ToString();

      if (!string.IsNullOrEmpty(originalId) && !string.IsNullOrEmpty(newId))
      {
          int originalIndex = -1;
          int newIndex = -1;

          for (int i = 0; i < dt.Rows.Count; i++)
          {
              string lasmplid = dt.Rows[i]["LASMPLID"].ToString();
              if (lasmplid == originalId)
                  originalIndex = i;
              if (lasmplid == newId)
                  newIndex = i;
          }

          if (originalIndex >= 0 && newIndex >= 0 && newIndex != originalIndex + 1)
          {
              // Copy data FIRST
              object[] rowData = dt.Rows[newIndex].ItemArray.Clone() as object[];

              // Remove old row
              dt.Rows.RemoveAt(newIndex);

              if (newIndex < originalIndex)
                  originalIndex--;

              // Insert copied row
              DataRow copyRow = dt.NewRow();
              copyRow.ItemArray = rowData;
              dt.Rows.InsertAt(copyRow, originalIndex + 1);
          }

          Session["OriginalLasmplid"] = null;
          Session["NewLasmplid"] = null;
      }

      GridView1.DataSource = dt;
      GridView1.DataBind();

      if (GridView1.HeaderRow != null)
      {
          GridView1.HeaderRow.Cells[3].Text = columnName;
      }
  }
---

 protected void GridView1_RowDataBound(object sender, GridViewRowEventArgs e)
 {
     if (e.Row.RowType == DataControlRowType.Header)
     {
         // Update dynamic column header
         string columnName = (Session["ColumnName"] as string) ?? "TEMP_CENT";
         e.Row.Cells[3].Text = columnName;
     }

     // Highlight the newly copied row
     if (e.Row.RowType == DataControlRowType.DataRow)
     {
         string lasmplid = DataBinder.Eval(e.Row.DataItem, "LASMPLID")?.ToString();
         string highlightId = Session["HighlightRowId"]?.ToString();

         if (!string.IsNullOrEmpty(highlightId) && lasmplid == highlightId)
         {
             e.Row.Style["background-color"] = "#ffffcc"; // Light yellow highlight
             Session["HighlightRowId"] = null; // Clear after highlighting
         }
     }
 }


----------------------------------------------------------------------------------------------------------------------------------------------------------
<form id="form1" runat="server">
        <div class="confirm-container">
            <div class="confirm-message">
                <h3>Copy Record Confirmation</h3>
                <p>Are you sure you want to copy this record?</p>
            </div>
            <div class="button-container">
                <asp:Button ID="btnCopy" runat="server" Text="Copy" CssClass="btn btn-primary" OnClick="btnCopy_Click" />
                <asp:Button ID="btnCancel" runat="server" Text="Cancel" CssClass="btn btn-secondary" OnClientClick="window.close(); return false;" />
            </div>
        </div>
    </form>


------------------------------------------------------------------------
  <asp:Button ID="btnCopy" runat="server" 
     Text="C" 
     CssClass="btn-copy"
     CommandName="CopyRecord"
     CommandArgument='<%# Eval("LASMPLID") %>'
     OnClientClick='<%# "openCopyPopup(" + Eval("LASMPLID") + "); return false;" %>' />

     --
     function openCopyPopup(lasmplid) {
    var width = 500;
    var height = 500;
    var left = (screen.width - width) / 2;
    var top = (screen.height - height) / 2;

    // Open popup with no toolbars, scrollbars, or resize capability
    window.open('CopyConfirm.aspx?id=' + lasmplid, 'CopyWindow',
        'width=' + width +
        ',height=' + height +
        ',left=' + left +
        ',top=' + top +
        ',toolbar=no' +
        ',location=no' +
        ',directories=no' +
        ',status=no' +
        ',menubar=no' +
        ',scrollbars=no' +
        ',resizable=no' +
        ',copyhistory=no');

    return false;
}

-------------------------------------------------------------------------------------------



<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="CopyConfirm.aspx.cs" Inherits="Report.CopyConfirm" %>

<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>Copy Confirmation</title>
   <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
            background-color: #f5f5f5;
        }

        .confirm-container {
            background-color: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            text-align: center;
        }

        .confirm-message {
            font-size: 16px;
            margin-bottom: 30px;
            color: #333;
        }

        .button-container {
            display: flex;
            justify-content: center;
            gap: 15px;
        }

        .btn {
            padding: 10px 30px;
            font-size: 14px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
        }

        .btn-primary {
            background-color: #007bff;
            color: white;
        }

        .btn-primary:hover {
            background-color: #0056b3;
        }

        .btn-secondary {
            background-color: #6c757d;
            color: white;
        }

        .btn-secondary:hover {
            background-color: #545b62;
        }
    </style>
    <script type="text/javascript">
        // Center the window on load
        window.onload = function() {
            // Remove window controls (works in some browsers)
            window.resizeTo(500, 350);
            
            // Center the window
            var width = 500;
            var height = 350;
            var left = (screen.width - width) / 2;
            var top = (screen.height - height) / 2;
            window.moveTo(left, top);
        };

        // Prevent context menu
        document.addEventListener('contextmenu', function(e) {
            e.preventDefault();
        });
    </script>
</head>
<body>
    <form id="form1" runat="server">
        <div class="confirm-container">
            <div class="confirm-message">
                <h3>Copy Record Confirmation</h3>
                <p>Are you sure you want to copy this record?</p>
            </div>
            <div class="button-container">
                <asp:Button ID="btnCopy" runat="server" Text="Copy" CssClass="btn btn-primary" OnClick="btnCopy_Click" />
                <asp:Button ID="btnCancel" runat="server" Text="Cancel" CssClass="btn btn-secondary" OnClientClick="window.close(); return false;" />
            </div>
        </div>
    </form>
</body>
</html>

----------------------------------------------------------------------------------------------------------------------------

using System;
using System.Data;
using System.Data.SqlClient;
using System.Web.UI;

namespace Report
{
    public partial class CopyConfirm : System.Web.UI.Page
    {
        private string connectionString = @"Server=localhost\SQLEXPRESS02;Initial Catalog=BEFS_ADMIN;Integrated Security=True";

        protected void Page_Load(object sender, EventArgs e)
        {
            if (!IsPostBack)
            {
                // Get the ID from query string
                string id = Request.QueryString["id"];
                if (!string.IsNullOrEmpty(id))
                {
                    Session["CopyId"] = id;
                }
            }
        }

        protected void btnCopy_Click(object sender, EventArgs e)
        {
            string id = Session["CopyId"]?.ToString();

            if (!string.IsNullOrEmpty(id))
            {
                CopyRecordInDb(id);

                // Close the popup and refresh parent window
                string script = @"
                    if (window.opener && !window.opener.closed) {
                        window.opener.location.reload();
                    }
                    window.close();";

                ScriptManager.RegisterStartupScript(this, GetType(), "closeWindow", script, true);
            }
        }

        private void CopyRecordInDb(string id)
        {

            try
            {
                using (SqlConnection conn = new SqlConnection(connectionString))
                {
                    conn.Open();

                    // Start a transaction
                    using (SqlTransaction transaction = conn.BeginTransaction())
                    {
                        try
                        {
                            // Get the maximum LASMPLID to generate new ID
                            string getMaxIdQuery = "SELECT ISNULL(MAX(LASMPLID), 0) + 1 FROM dbo.SC_CHM_ALL";
                            long newLasmplid;

                            using (SqlCommand cmdMaxId = new SqlCommand(getMaxIdQuery, conn, transaction))
                            {
                                newLasmplid = Convert.ToInt64(cmdMaxId.ExecuteScalar());
                            }

                            // Copy the record from SC_CHM_ALL
                            string copySCChmQuery = @"
                                INSERT INTO dbo.SC_CHM_ALL (
                                    PROG_CODE, SITE_NAME, SITEID_NO, SITE_CNTY, HUC8, BASIN, SITE_LOCT,
                                    COL_DATEX, COL_DATE, COL_TIME, ACETOCHLR, ACETOCHLRR, ALACHLOR, ALACHLORR,
                                    ALDRIN, ALDRINR, ALKALINTY, ALKALINTYR, A_BHC, A_BHCR, ALUMINUM, ALUMINUMR,
                                    AMMONIA, AMMONIAR, ANTIMONY, ANTIMONYR, ARSENIC, ARSENICR, ATRAZIN, ATRAZINR,
                                    BARIUM, BARIUMR, BERYLLIUM, BERYLLIUMR, B_BHC, B_BHCR, BOD, BODR, BORON, BORONR,
                                    BROMIDE, BROMIDER, BUTACHLOR, BUTACHLORR, CADMIUM, CADMIUMR, CALCIUM, CALCIUMR,
                                    CARBOFURN, CARBOFURNR, CHLORDANE, CHLORDANER, CHLORIDE, CHLORIDER, CHROMIUM, CHROMIUMR,
                                    COBALT, COBALTR, COPPER, COPPERR, CYANAZINE, CYANAZINER, DACTHAL, DACTHALR,
                                    DEEATRAZN, DEEATRAZNR, DISATRAZN, DISATRAZNR, D_BHC, D_BHCR, DIAZINON, DIAZINONR,
                                    DIELDRIN, DIELDRINR, DISOXY, DISOXYR, ENDOSULF1, ENDOSULF1R, ENDOSULF2, ENDOSULF2R,
                                    ENDRIN, ENDRINR, FECCOLI, FECCOLIR, FECSTRP, FECSTRPR, FLUORIDE, FLUORIDER,
                                    LINDANE, LINDANER, HEPTCHLR, HEPTCHLRR, HEPTCHLRE, HEPTCHLRER, HEPTCHLRB, HEPTCHLRBR,
                                    IRON, IRONR, LEAD, LEADR, MAGNESIUM, MAGNESIUMR, MANGANESE, MANGANESER,
                                    MERCURY, MERCURYR, METOCLR, METOCLRR, MTRBUZI, MTRBUZIR, MOLYBDENM, MOLYBDENMR,
                                    NICKEL, NICKELR, NITRATE, NITRATER, NITRITE, NITRITER, NO2_NO3, NO2_NO3R,
                                    ORTH_PHOS, ORTH_PHOSR, PP_DDD, PP_DDDR, PP_DDE, PP_DDER, PP_DDT, PP_DDTR,
                                    PCB_1016, PCB_1016R, PCB_1221, PCB_1221R, PCB_1232, PCB_1232R, PCB_1242, PCB_1242R,
                                    PCB_1248, PCB_1248R, PCB_1254, PCB_1254R, PCB_1260, PCB_1260R, PHLAB, PHLABR,
                                    PHFIELD, PHFIELDR, PICLORAM, PICLORAMR, POTTASIUM, POTTASIUMR, PROMETON, PROMETONR,
                                    PROPACHLR, PROPACHLRR, PROPAZINE, PROPAZINER, SELENIUM, SELENIUMR, SILICA, SILICAR,
                                    SILVER, SILVERR, SIMAZINE, SIMAZINER, SODIUM, SODIUMR, SPEC_COND, SPEC_CONDR,
                                    SULFATE, SULFATER, TEMP_CENT, TEMP_CENTR, THALLIUM, THALLIUMR, TDS, TDSR,
                                    TOTHARD, TOTHARDR, PHOSPHU, PHOSPHUR, TSS, TSSR, TOXAPHENE, TOXAPHENER,
                                    TURBIDITY, TURBIDITYR, VANADIUM, VANADIUMR, X24D, X24DR, X245T, X245TR,
                                    ZINC, ZINCR, LABNUM, LASMPLID, HCCP, HCCPR, X245TP, X245TPR, DURSBAN, DURSBANR,
                                    TOC, TOCR, KJELDAHL, KJELDAHLR, STRONTIUM, STRONTIUMR, MTHOXYCHL, MTHOXYCHLR,
                                    ENDOSULFS, ENDOSULFSR, DSTAMP, CHLOROPH, CHLOROPHR, ECOLI, ECOLIR, BROMACIL, BROMACILR,
                                    PCP, PCPR, COD, CODR, CHARD, CHARDR, OPDDT, OPDDTR, PCBS, PCBSR, ZOOPLANK, ZOOPLANKR,
                                    TSTAMP, DEPTH, TOTCOLI, TOTCOLIR, SECCHI, SECCHIR, PAR, PARR, ENTERO, ENTEROR,
                                    URANIUM, URANIUMR, TURBFLD, TURBFLDR, DISOXYFLD, DISOXYFLDR, CONDFLD, CONDFLDR, NUM_PARMS
                                )
                                SELECT 
                                    PROG_CODE, SITE_NAME, SITEID_NO, SITE_CNTY, HUC8, BASIN, SITE_LOCT,
                                    COL_DATEX, COL_DATE, COL_TIME, ACETOCHLR, ACETOCHLRR, ALACHLOR, ALACHLORR,
                                    ALDRIN, ALDRINR, ALKALINTY, ALKALINTYR, A_BHC, A_BHCR, ALUMINUM, ALUMINUMR,
                                    AMMONIA, AMMONIAR, ANTIMONY, ANTIMONYR, ARSENIC, ARSENICR, ATRAZIN, ATRAZINR,
                                    BARIUM, BARIUMR, BERYLLIUM, BERYLLIUMR, B_BHC, B_BHCR, BOD, BODR, BORON, BORONR,
                                    BROMIDE, BROMIDER, BUTACHLOR, BUTACHLORR, CADMIUM, CADMIUMR, CALCIUM, CALCIUMR,
                                    CARBOFURN, CARBOFURNR, CHLORDANE, CHLORDANER, CHLORIDE, CHLORIDER, CHROMIUM, CHROMIUMR,
                                    COBALT, COBALTR, COPPER, COPPERR, CYANAZINE, CYANAZINER, DACTHAL, DACTHALR,
                                    DEEATRAZN, DEEATRAZNR, DISATRAZN, DISATRAZNR, D_BHC, D_BHCR, DIAZINON, DIAZINONR,
                                    DIELDRIN, DIELDRINR, DISOXY, DISOXYR, ENDOSULF1, ENDOSULF1R, ENDOSULF2, ENDOSULF2R,
                                    ENDRIN, ENDRINR, FECCOLI, FECCOLIR, FECSTRP, FECSTRPR, FLUORIDE, FLUORIDER,
                                    LINDANE, LINDANER, HEPTCHLR, HEPTCHLRR, HEPTCHLRE, HEPTCHLRER, HEPTCHLRB, HEPTCHLRBR,
                                    IRON, IRONR, LEAD, LEADR, MAGNESIUM, MAGNESIUMR, MANGANESE, MANGANESER,
                                    MERCURY, MERCURYR, METOCLR, METOCLRR, MTRBUZI, MTRBUZIR, MOLYBDENM, MOLYBDENMR,
                                    NICKEL, NICKELR, NITRATE, NITRATER, NITRITE, NITRITER, NO2_NO3, NO2_NO3R,
                                    ORTH_PHOS, ORTH_PHOSR, PP_DDD, PP_DDDR, PP_DDE, PP_DDER, PP_DDT, PP_DDTR,
                                    PCB_1016, PCB_1016R, PCB_1221, PCB_1221R, PCB_1232, PCB_1232R, PCB_1242, PCB_1242R,
                                    PCB_1248, PCB_1248R, PCB_1254, PCB_1254R, PCB_1260, PCB_1260R, PHLAB, PHLABR,
                                    PHFIELD, PHFIELDR, PICLORAM, PICLORAMR, POTTASIUM, POTTASIUMR, PROMETON, PROMETONR,
                                    PROPACHLR, PROPACHLRR, PROPAZINE, PROPAZINER, SELENIUM, SELENIUMR, SILICA, SILICAR,
                                    SILVER, SILVERR, SIMAZINE, SIMAZINER, SODIUM, SODIUMR, SPEC_COND, SPEC_CONDR,
                                    SULFATE, SULFATER, TEMP_CENT, TEMP_CENTR, THALLIUM, THALLIUMR, TDS, TDSR,
                                    TOTHARD, TOTHARDR, PHOSPHU, PHOSPHUR, TSS, TSSR, TOXAPHENE, TOXAPHENER,
                                    TURBIDITY, TURBIDITYR, VANADIUM, VANADIUMR, X24D, X24DR, X245T, X245TR,
                                    ZINC, ZINCR, LABNUM, @newLasmplid, HCCP, HCCPR, X245TP, X245TPR, DURSBAN, DURSBANR,
                                    TOC, TOCR, KJELDAHL, KJELDAHLR, STRONTIUM, STRONTIUMR, MTHOXYCHL, MTHOXYCHLR,
                                    ENDOSULFS, ENDOSULFSR, GETDATE(), CHLOROPH, CHLOROPHR, ECOLI, ECOLIR, BROMACIL, BROMACILR,
                                    PCP, PCPR, COD, CODR, CHARD, CHARDR, OPDDT, OPDDTR, PCBS, PCBSR, ZOOPLANK, ZOOPLANKR,
                                    GETDATE(), DEPTH, TOTCOLI, TOTCOLIR, SECCHI, SECCHIR, PAR, PARR, ENTERO, ENTEROR,
                                    URANIUM, URANIUMR, TURBFLD, TURBFLDR, DISOXYFLD, DISOXYFLDR, CONDFLD, CONDFLDR, NUM_PARMS
                                FROM dbo.SC_CHM_ALL
                                WHERE LASMPLID = @oldId";

                            using (SqlCommand cmdCopy = new SqlCommand(copySCChmQuery, conn, transaction))
                            {
                                cmdCopy.Parameters.AddWithValue("@newLasmplid", newLasmplid);
                                cmdCopy.Parameters.AddWithValue("@oldId", id);
                                cmdCopy.ExecuteNonQuery();
                            }

                            // Copy from SCIONS table if exists
                            //string copyScionsQuery = @"
                            //    IF EXISTS (SELECT 1 FROM BEFS_ADMIN.SCIONS WHERE LASMPLID = @oldId)
                            //    BEGIN
                            //        INSERT INTO dbo.SCIONS (LASMPLID, ION_NAME, ION_VALUE, ION_REMARK)
                            //        SELECT @newLasmplid, ION_NAME, ION_VALUE, ION_REMARK
                            //        FROM dbo.SCIONS
                            //        WHERE LASMPLID = @oldId
                            //    END";

                            //using (SqlCommand cmdCopyIons = new SqlCommand(copyScionsQuery, conn, transaction))
                            //{
                            //    cmdCopyIons.Parameters.AddWithValue("@newLasmplid", newLasmplid);
                            //    cmdCopyIons.Parameters.AddWithValue("@oldId", id);
                            //    cmdCopyIons.ExecuteNonQuery();
                            //}

                            // Commit the transaction
                            transaction.Commit();
                        }
                        catch (Exception ex)
                        {
                            // Rollback on error
                            transaction.Rollback();
                            throw new Exception($"Error copying record: {ex.Message}", ex);
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine($"Copy error: {ex.Message}");
                // You might want to show an error message to the user here
            }
        }
    }
}
--------------------------------------------------------------------------------------


 
 ------------------------------------------------------------------------------------------------------------------------------------
 private bool UpsertRecord(string tableName,string idColumnName,object idValue,Dictionary<string, object> columnData)
 {
     if (columnData == null || columnData.Count == 0)
     {
         ShowMessage("No data to save.", false);
         return false;
     }

     using (SqlConnection conn = new SqlConnection(connectionString))
     {
         try
         {
             conn.Open();

             // ============================
             // 1️⃣ CHECK IF RECORD EXISTS
             // ============================
             string existsQuery =
                 $"SELECT COUNT(1) FROM {tableName} WHERE {idColumnName} = @IdValue";

             bool recordExists;
             using (SqlCommand existsCmd = new SqlCommand(existsQuery, conn))
             {
                 existsCmd.Parameters.AddWithValue("@IdValue", idValue);
                 recordExists = Convert.ToInt32(existsCmd.ExecuteScalar()) > 0;
             }

             // ============================
             // 2️⃣ UPDATE IF EXISTS
             // ============================
             if (recordExists)
             {
                 StringBuilder updateQuery = new StringBuilder();
                 updateQuery.Append($"UPDATE {tableName} SET ");

                 List<string> setStatements = new List<string>();
                 foreach (var column in columnData.Keys)
                     setStatements.Add($"{column} = @{column}");

                 updateQuery.Append(string.Join(", ", setStatements));
                 updateQuery.Append($" WHERE {idColumnName} = @IdValue");

                 using (SqlCommand updateCmd = new SqlCommand(updateQuery.ToString(), conn))
                 {
                     updateCmd.Parameters.AddWithValue("@IdValue", idValue);

                     foreach (var kvp in columnData)
                         updateCmd.Parameters.AddWithValue($"@{kvp.Key}", kvp.Value ?? DBNull.Value);

                     return updateCmd.ExecuteNonQuery() > 0;
                 }
             }

             // ============================
             // 3️⃣ INSERT IF NOT EXISTS
             // ============================
             columnData[idColumnName] = idValue;

             string columns = string.Join(", ", columnData.Keys);
             string parameters = string.Join(", ", columnData.Keys.Select(c => "@" + c));

             string insertQuery =
                 $"INSERT INTO {tableName} ({columns}) VALUES ({parameters})";

             using (SqlCommand insertCmd = new SqlCommand(insertQuery, conn))
             {
                 foreach (var kvp in columnData)
                     insertCmd.Parameters.AddWithValue($"@{kvp.Key}", kvp.Value ?? DBNull.Value);

                 return insertCmd.ExecuteNonQuery() > 0;
             }
         }
         catch (Exception ex)
         {
             ShowMessage("Error saving record: " + ex.Message, false);
             return false;
         }
     }
 }



















FOR UPDATING DATA
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
using Microsoft.Ajax.Utilities;
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Data.SqlClient;
using System.Linq;
using System.Text;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;

namespace Report
{
    //public partial class ChemicalData : System.Web.UI.Page
    //{
    //    protected void Page_Load(object sender, EventArgs e)
    //    {

    //    }
    //}

    public partial class ChemicalData : System.Web.UI.Page
    {
        private string connectionString = @"Server=localhost\SQLEXPRESS02;Initial Catalog=ChemicalAnalysisDB;Integrated Security=True";

        protected void Page_Load(object sender, EventArgs e)
        {
            if (!IsPostBack)
            {
                LoadData();
            }
        }

        private void LoadData()
        {
            Session["id"] = 1;
            if (Session["id"] == null)
            {
                ShowMessage("Session ID not found. Please login again.", false);
                return;
            }
            int recordId = Convert.ToInt32(Session["id"]);
            using (SqlConnection conn = new SqlConnection(connectionString))
            {
                try
                {
                    conn.Open();
                    string query = @"SELECT * FROM ChemicalData WHERE Id = @Id";

                    using (SqlCommand cmd = new SqlCommand(query, conn))
                    {
                        cmd.Parameters.AddWithValue("@Id", recordId);

                        using (SqlDataReader reader = cmd.ExecuteReader())
                        {
                            if (reader.Read())
                            {
                                // Populate all textboxes with data from database
                                txtAlkalinity.Text = GetSafeString(reader, "Alkalinity");
                                txtAluminum.Text = GetSafeString(reader, "Aluminum");
                                txtAntimony.Text = GetSafeString(reader, "Antimony");
                                txtArsenic.Text = GetSafeString(reader, "Arsenic");
                                txtBarium.Text = GetSafeString(reader, "Barium");
                                txtBeryllium.Text = GetSafeString(reader, "Beryllium");
                                txtBOD.Text = GetSafeString(reader, "BOD");
                                txtBoron.Text = GetSafeString(reader, "Boron");
                                txtBromide.Text = GetSafeString(reader, "Bromide");
                                txtCadmium.Text = GetSafeString(reader, "Cadmium");
                                txtCalcium.Text = GetSafeString(reader, "Calcium");
                                txtChloride.Text = GetSafeString(reader, "Chloride");
                                txtChloroph.Text = GetSafeString(reader, "Chloroph");
                                txtChromium.Text = GetSafeString(reader, "Chromium");
                                txtCobalt.Text = GetSafeString(reader, "Cobalt");
                                txtFieldnotes.Text = GetSafeString(reader, "Fieldnotes");
                                txtSampledate.Text = GetSafeString(reader, "Sampledate");
                                txtCollectors.Text = GetSafeString(reader, "Collectors");
                                txtCOD.Text = GetSafeString(reader, "COD");
                                txtCopper.Text = GetSafeString(reader, "Copper");
                                txtDisOxy.Text = GetSafeString(reader, "DisOxy");
                                txtEColi.Text = GetSafeString(reader, "EColi");
                                txtEntero.Text = GetSafeString(reader, "Entero");
                                txtFecColi.Text = GetSafeString(reader, "FecColi");
                                txtFecStrp.Text = GetSafeString(reader, "FecStrp");
                                txtFluride.Text = GetSafeString(reader, "Fluride");
                                txtIron.Text = GetSafeString(reader, "Iron");
                                txtKjeldahl.Text = GetSafeString(reader, "Kjeldahl");
                                txtLead.Text = GetSafeString(reader, "Lead");
                                txtMagnesium.Text = GetSafeString(reader, "Magnesium");
                                txtManganese.Text = GetSafeString(reader, "Manganese");
                                txtMercury.Text = GetSafeString(reader, "Mercury");
                                txtMolybdenum.Text = GetSafeString(reader, "Molybdenum");
                                txtNickel.Text = GetSafeString(reader, "Nickel");
                                txtNitrate.Text = GetSafeString(reader, "Nitrate");
                                txtNO2ANO3.Text = GetSafeString(reader, "NO2ANO3");
                                txtOrthPhos.Text = GetSafeString(reader, "OrthPhos");
                                txtPar.Text = GetSafeString(reader, "Par");
                                txtPHField.Text = GetSafeString(reader, "PHField");
                                txtPHLab.Text = GetSafeString(reader, "PHLab");
                                txtPotassium.Text = GetSafeString(reader, "Potassium");
                                txtSecchi.Text = GetSafeString(reader, "Secchi");
                                txtSelenium.Text = GetSafeString(reader, "Selenium");
                                txtSilica.Text = GetSafeString(reader, "Silica");
                                txtSilver.Text = GetSafeString(reader, "Silver");
                                txtSodium.Text = GetSafeString(reader, "Sodium");
                                txtSpecCondFic.Text = GetSafeString(reader, "SpecCondFic");
                                txtSpecCond.Text = GetSafeString(reader, "SpecCond");
                                txtLabnum.Text = GetSafeString(reader, "Labnum");
                                txtStrontium.Text = GetSafeString(reader, "Strontium");
                                txtNitrite.Text = GetSafeString(reader, "Nitrite");
                                txtTempCast.Text = GetSafeString(reader, "TempCast");
                                txtThallium.Text = GetSafeString(reader, "Thallium");
                                txtTOC.Text = GetSafeString(reader, "TOC");
                                txtTotColi.Text = GetSafeString(reader, "TotColi");
                                txtTotHard.Text = GetSafeString(reader, "TotHard");
                                txtTotNierd.Text = GetSafeString(reader, "TotNierd");
                                txtTSS.Text = GetSafeString(reader, "TSS");
                                txtTurbidityFid.Text = GetSafeString(reader, "TurbidityFid");
                                txtSulfate.Text = GetSafeString(reader, "Sulfate");
                                txtTurbidity.Text = GetSafeString(reader, "Turbidity");
                                txtZinc.Text = GetSafeString(reader, "Zinc");
                                txtUranium.Text = GetSafeString(reader, "Uranium");
                                txtSava.Text = GetSafeString(reader, "Sava");
                            }
                            else
                            {
                                ShowMessage("No record found for the given ID.", false);
                            }
                        }
                    }
                }
                catch (Exception ex)
                {
                    ShowMessage("Error loading data: " + ex.Message, false);
                }
            }
        }

        protected void btnSave_Click(object sender, EventArgs e)
        {
            if (Session["id"] == null)
            {
                ShowMessage("Session ID not found. Please login again.", false);
                return;
            }

            int recordId = Convert.ToInt32(Session["id"]);

            // Prepare the column data dictionary
            Dictionary<string, object> columnData = new Dictionary<string, object>
            {
                { "Alkalinity", GetTextBoxValue(txtAlkalinity.Text) },
                { "Aluminum", GetTextBoxValue(txtAluminum.Text) },
                { "Antimony", GetTextBoxValue(txtAntimony.Text) },
                { "Arsenic", GetTextBoxValue(txtArsenic.Text) },
                { "Barium", GetTextBoxValue(txtBarium.Text) },
                { "Beryllium", GetTextBoxValue(txtBeryllium.Text) },
                { "BOD", GetTextBoxValue(txtBOD.Text) },
                { "Boron", GetTextBoxValue(txtBoron.Text) },
                { "Bromide", GetTextBoxValue(txtBromide.Text) },
                { "Cadmium", GetTextBoxValue(txtCadmium.Text) },
                { "Calcium", GetTextBoxValue(txtCalcium.Text) },
                { "Chloride", GetTextBoxValue(txtChloride.Text) },
                { "Chloroph", GetTextBoxValue(txtChloroph.Text) },
                { "Chromium", GetTextBoxValue(txtChromium.Text) },
                { "Cobalt", GetTextBoxValue(txtCobalt.Text) },
                { "Fieldnotes", GetTextBoxValue(txtFieldnotes.Text) },
                { "Sampledate", GetTextBoxValue(txtSampledate.Text) },
                { "Collectors", GetTextBoxValue(txtCollectors.Text) },
                { "COD", GetTextBoxValue(txtCOD.Text) },
                { "Copper", GetTextBoxValue(txtCopper.Text) },
                { "DisOxy", GetTextBoxValue(txtDisOxy.Text) },
                { "EColi", GetTextBoxValue(txtEColi.Text) },
                { "Entero", GetTextBoxValue(txtEntero.Text) },
                { "FecColi", GetTextBoxValue(txtFecColi.Text) },
                { "FecStrp", GetTextBoxValue(txtFecStrp.Text) },
                { "Fluride", GetTextBoxValue(txtFluride.Text) },
                { "Iron", GetTextBoxValue(txtIron.Text) },
                { "Kjeldahl", GetTextBoxValue(txtKjeldahl.Text) },
                { "Lead", GetTextBoxValue(txtLead.Text) },
                { "Magnesium", GetTextBoxValue(txtMagnesium.Text) },
                { "Manganese", GetTextBoxValue(txtManganese.Text) },
                { "Mercury", GetTextBoxValue(txtMercury.Text) },
                { "Molybdenum", GetTextBoxValue(txtMolybdenum.Text) },
                { "Nickel", GetTextBoxValue(txtNickel.Text) },
                { "Nitrate", GetTextBoxValue(txtNitrate.Text) },
                { "NO2ANO3", GetTextBoxValue(txtNO2ANO3.Text) },
                { "OrthPhos", GetTextBoxValue(txtOrthPhos.Text) },
                { "Par", GetTextBoxValue(txtPar.Text) },
                { "PHField", GetTextBoxValue(txtPHField.Text) },
                { "PHLab", GetTextBoxValue(txtPHLab.Text) },
                { "Potassium", GetTextBoxValue(txtPotassium.Text) },
                { "Secchi", GetTextBoxValue(txtSecchi.Text) },
                { "Selenium", GetTextBoxValue(txtSelenium.Text) },
                { "Silica", GetTextBoxValue(txtSilica.Text) },
                { "Silver", GetTextBoxValue(txtSilver.Text) },
                { "Sodium", GetTextBoxValue(txtSodium.Text) },
                { "SpecCondFic", GetTextBoxValue(txtSpecCondFic.Text) },
                { "SpecCond", GetTextBoxValue(txtSpecCond.Text) },
                { "Labnum", GetTextBoxValue(txtLabnum.Text) },
                { "Strontium", GetTextBoxValue(txtStrontium.Text) },
                { "Nitrite", GetTextBoxValue(txtNitrite.Text) },
                { "TempCast", GetTextBoxValue(txtTempCast.Text) },
                { "Thallium", GetTextBoxValue(txtThallium.Text) },
                { "TOC", GetTextBoxValue(txtTOC.Text) },
                { "TotColi", GetTextBoxValue(txtTotColi.Text) },
                { "TotHard", GetTextBoxValue(txtTotHard.Text) },
                { "TotNierd", GetTextBoxValue(txtTotNierd.Text) },
                { "TSS", GetTextBoxValue(txtTSS.Text) },
                { "TurbidityFid", GetTextBoxValue(txtTurbidityFid.Text) },
                { "Sulfate", GetTextBoxValue(txtSulfate.Text) },
                { "Turbidity", GetTextBoxValue(txtTurbidity.Text) },
                { "Zinc", GetTextBoxValue(txtZinc.Text) },
                { "Uranium", GetTextBoxValue(txtUranium.Text) },
                { "Sava", GetTextBoxValue(txtSava.Text) }
            };

            // Call the dynamic update function
            bool success = UpdateRecord("ChemicalData", "Id", recordId, columnData);

            if (success)
            {
                ShowMessage("Data saved successfully!", true);
            }
            else
            {
                ShowMessage("Error saving data. Please check the logs.", false);
            }
        }

        private bool UpdateRecord(string tableName, string idColumnName, object idValue, Dictionary<string, object> columnData)
        {
            if (columnData == null || columnData.Count == 0)
            {
                ShowMessage("No data to update.", false);
                return false;
            }

            using (SqlConnection conn = new SqlConnection(connectionString))
            {
                try
                {
                    conn.Open();
                    StringBuilder queryBuilder = new StringBuilder();
                    queryBuilder.Append($"UPDATE {tableName} SET ");
                    List<string> setStatements = new List<string>();
                    foreach (var column in columnData.Keys)
                        setStatements.Add($"{column} = @{column}");
                    queryBuilder.Append(string.Join(", ", setStatements));
                    queryBuilder.Append($" WHERE {idColumnName} = @IdValue");
                    string query = queryBuilder.ToString();
                    using (SqlCommand cmd = new SqlCommand(query, conn))
                    {
                        cmd.Parameters.AddWithValue("@IdValue", idValue);
                        foreach (var kvp in columnData)
                            cmd.Parameters.AddWithValue($"@{kvp.Key}", kvp.Value ?? DBNull.Value);
                        int rowsAffected = cmd.ExecuteNonQuery();
                        return rowsAffected > 0;
                    }
                }
                catch (Exception ex)
                {
                    ShowMessage("Error updating record: " + ex.Message, false);
                    return false;
                }
            }
        }

        private string GetSafeString(SqlDataReader reader, string columnName)
        {
            int ordinal = reader.GetOrdinal(columnName);
            return reader.IsDBNull(ordinal) ? string.Empty : reader.GetString(ordinal);
        }

        private object GetTextBoxValue(string text)
        {
            return string.IsNullOrWhiteSpace(text) ? (object)DBNull.Value : text.Trim();
        }

        private void ShowMessage(string message, bool isSuccess)
        {
            lblMessage.Text = message;
            lblMessage.CssClass = isSuccess ? "message success" : "message error";
        }
    }
}
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
public static int UpdateRow(
    SqlConnection connection,
    DataRow row,
    string tableName,
    int id)
{
    if (row == null)
        throw new ArgumentNullException(nameof(row));

    if (string.IsNullOrWhiteSpace(tableName))
        throw new ArgumentException("Table name is required");

    List<string> setClauses = new List<string>();
    SqlCommand cmd = new SqlCommand();
    cmd.Connection = connection;

    foreach (DataColumn column in row.Table.Columns)
    {
        // Skip primary key
        if (column.ColumnName.Equals("Id", StringComparison.OrdinalIgnoreCase))
            continue;

        string paramName = "@" + column.ColumnName;

        setClauses.Add($"{column.ColumnName} = {paramName}");

        object value = row[column.ColumnName] ?? DBNull.Value;
        cmd.Parameters.AddWithValue(paramName, value);
    }

    if (setClauses.Count == 0)
        throw new InvalidOperationException("No columns to update");

    cmd.Parameters.Add("@Id", SqlDbType.Int).Value = id;

    cmd.CommandText = $@"
        UPDATE {tableName}
        SET {string.Join(", ", setClauses)}
        WHERE Id = @Id";

    if (connection.State != ConnectionState.Open)
        connection.Open();

    return cmd.ExecuteNonQuery();
}





Update the row 
---------------------------

DataRow row = dataTable.Rows[0];

using (SqlConnection con = new SqlConnection(connectionString))
{
    string sql = @"UPDATE Employee
                   SET Name = @Name,
                       Age = @Age,
                       Salary = @Salary
                   WHERE Id = @Id";

    using (SqlCommand cmd = new SqlCommand(sql, con))
    {
        cmd.Parameters.Add("@Name", SqlDbType.VarChar).Value = row["Name"];
        cmd.Parameters.Add("@Age", SqlDbType.Int).Value = row["Age"];
        cmd.Parameters.Add("@Salary", SqlDbType.Decimal).Value = row["Salary"];
        cmd.Parameters.Add("@Id", SqlDbType.Int).Value = row["Id"]; // Primary Key

        con.Open();
        cmd.ExecuteNonQuery();
    }
}



--------------------------------------------------------------------------------------------------
Correct line of code for thickness
-----------------------------------------

 Font messageFont = new Font(bf, 10, Font.NORMAL);
 Chunk messageChunk = new Chunk(row["Message"].ToString(), messageFont);

 // thickness = underline thickness
 // yOffset   = move underline DOWN (critical)
 messageChunk.SetUnderline(0.6f, -1f);

 Paragraph para = new Paragraph
 {
     Leading = 18f   // line height
 };

 para.Add(messageChunk);

 cell = new PdfPCell(para)
 {
     Border = Rectangle.NO_BORDER,
     PaddingTop = 3,
     PaddingBottom = 8,
     VerticalAlignment = Element.ALIGN_TOP
 };

 table.AddCell(cell);



---------------------------------------------------------------------------------------------

FOR CLOSING POP Window
----------------------------------

protected void btnSaveAndClose_Click(object sender, EventArgs e)
{
    // Perform server-side actions here (e.g., save data to database)
    // SaveData(); 

    // Inject script to close the popup AFTER the page reloads on the client
    string script = "closePopup();"; // Reusing the JavaScript function from Step 1

    // Use ScriptManager if you have one, useful with UpdatePanels
    if (ScriptManager.GetCurrent(this.Page) != null)
    {
        ScriptManager.RegisterStartupScript(this.Page, this.GetType(), "closeModalScript", script, true);
    }
    else // Standard Web Forms page
    {
        ClientScript.RegisterStartupScript(this.GetType(), "closeModalScript", script, true);
    }
}



-------------------------------------------------------------------------------------------------

For the time being Just Copy and paste below Private method in above class the one which had shared in above screen shot .
----------------------------------------------------------------------------------------------------------------------------
private byte[] GeneratePdfReport(DataTable data)
    {
        // Create instance report
        var report = new Telerik.Reporting.Report();
        report.PageSettings.Margins = new Telerik.Reporting.Drawing.MarginsU(
            new Telerik.Reporting.Drawing.Unit(0.5, UnitType.Inch));

        // Add report header
        var headerSection = new Telerik.Reporting.ReportHeaderSection();
        headerSection.Height = new Telerik.Reporting.Drawing.Unit(0.8, UnitType.Inch);

        var titleTextBox = new Telerik.Reporting.TextBox();
        titleTextBox.Value = "Water Quality Report";
        titleTextBox.Style.Font.Size = new Telerik.Reporting.Drawing.Unit(16, UnitType.Point);
        titleTextBox.Style.Font.Bold = true;
        titleTextBox.Style.TextAlign = HorizontalAlign.Center;
        titleTextBox.Location = new Telerik.Reporting.Drawing.PointU(
            new Telerik.Reporting.Drawing.Unit(0, UnitType.Inch), 
            new Telerik.Reporting.Drawing.Unit(0.1, UnitType.Inch));
        titleTextBox.Size = new Telerik.Reporting.Drawing.SizeU(
            new Telerik.Reporting.Drawing.Unit(7.5, UnitType.Inch), 
            new Telerik.Reporting.Drawing.Unit(0.4, UnitType.Inch));

        headerSection.Items.Add(titleTextBox);
        report.Items.Add(headerSection);

        // Add detail section with table
        var detailSection = new Telerik.Reporting.DetailSection();
        detailSection.Height = new Telerik.Reporting.Drawing.Unit(0.25, UnitType.Inch);

        var table = new Telerik.Reporting.Table();
        table.DataSource = data;
        table.Location = new Telerik.Reporting.Drawing.PointU(
            new Telerik.Reporting.Drawing.Unit(0, UnitType.Inch), 
            new Telerik.Reporting.Drawing.Unit(0, UnitType.Inch));
        table.Size = new Telerik.Reporting.Drawing.SizeU(
            new Telerik.Reporting.Drawing.Unit(7.5, UnitType.Inch), 
            new Telerik.Reporting.Drawing.Unit(0.5, UnitType.Inch));

        // Define table structure
        table.Body.Columns.Add(new Telerik.Reporting.TableBodyColumn(
            new Telerik.Reporting.Drawing.Unit(1, UnitType.Inch)));
        table.Body.Columns.Add(new Telerik.Reporting.TableBodyColumn(
            new Telerik.Reporting.Drawing.Unit(1.5, UnitType.Inch)));
        table.Body.Columns.Add(new Telerik.Reporting.TableBodyColumn(
            new Telerik.Reporting.Drawing.Unit(5, UnitType.Inch)));

        table.Body.Rows.Add(new Telerik.Reporting.TableBodyRow(
            new Telerik.Reporting.Drawing.Unit(0.25, UnitType.Inch)));

        // Column headers
        var headerRow = new Telerik.Reporting.TableGroup();
        table.RowGroups.Add(headerRow);
        headerRow.Name = "HeaderGroup";

        var typeHeader = new Telerik.Reporting.TextBox();
        typeHeader.Value = "Type";
        typeHeader.Style.Font.Bold = true;
        typeHeader.Style.BackgroundColor = System.Drawing.Color.LightGray;
        typeHeader.Style.BorderStyle.Default = BorderType.Solid;
        typeHeader.Style.BorderWidth.Default = new Telerik.Reporting.Drawing.Unit(1, UnitType.Pixel);

        var siteHeader = new Telerik.Reporting.TextBox();
        siteHeader.Value = "Site";
        siteHeader.Style.Font.Bold = true;
        siteHeader.Style.BackgroundColor = System.Drawing.Color.LightGray;
        siteHeader.Style.BorderStyle.Default = BorderType.Solid;
        siteHeader.Style.BorderWidth.Default = new Telerik.Reporting.Drawing.Unit(1, UnitType.Pixel);

        var messageHeader = new Telerik.Reporting.TextBox();
        messageHeader.Value = "Message";
        messageHeader.Style.Font.Bold = true;
        messageHeader.Style.BackgroundColor = System.Drawing.Color.LightGray;
        messageHeader.Style.BorderStyle.Default = BorderType.Solid;
        messageHeader.Style.BorderWidth.Default = new Telerik.Reporting.Drawing.Unit(1, UnitType.Pixel);

        // Data cells
        var typeCell = new Telerik.Reporting.TextBox();
        typeCell.Value = "=Fields.Type";
        typeCell.Style.BorderStyle.Default = BorderType.Solid;
        typeCell.Style.BorderWidth.Default = new Telerik.Reporting.Drawing.Unit(1, UnitType.Pixel);

        var siteCell = new Telerik.Reporting.TextBox();
        siteCell.Value = "=Fields.Site";
        siteCell.Style.BorderStyle.Default = BorderType.Solid;
        siteCell.Style.BorderWidth.Default = new Telerik.Reporting.Drawing.Unit(1, UnitType.Pixel);

        var messageCell = new Telerik.Reporting.TextBox();
        messageCell.Value = "=Fields.Message";
        messageCell.Style.BorderStyle.Default = BorderType.Solid;
        messageCell.Style.BorderWidth.Default = new Telerik.Reporting.Drawing.Unit(1, UnitType.Pixel);

        table.Body.SetCellContent(0, 0, typeCell);
        table.Body.SetCellContent(0, 1, siteCell);
        table.Body.SetCellContent(0, 2, messageCell);

        detailSection.Items.Add(table);
        report.Items.Add(detailSection);

        // Render to PDF
        var reportProcessor = new ReportProcessor();
        var result = reportProcessor.RenderReport("PDF", report, null);

        return result.DocumentBytes;
    }
    
    ----------------------------------------------------------------------------------------------------------------
    
    Then copy paste below code in you report button click.
    ---------------------------------------------------------------
    
    try
        {
            // Get data from database
            DataTable dt = GetScurptData();

            if (dt != null && dt.Rows.Count > 0)
            {
                // Generate PDF
                byte[] pdfBytes = GeneratePdfReport(dt);

                // Clear response
                Response.Clear();
                Response.ContentType = "application/pdf";
                Response.AddHeader("Content-Disposition", "inline; filename=WaterQualityReport.pdf");
                Response.BinaryWrite(pdfBytes);
                Response.End();
            }
            else
            {
                // Show message if no data
                ClientScript.RegisterStartupScript(this.GetType(), "alert", 
                    "alert('No data found for the report.');", true);
            }
        }
        catch (Exception ex)
        {
            // Log error and show message
            ClientScript.RegisterStartupScript(this.GetType(), "alert", 
                $"alert('Error generating report: {ex.Message}');", true);
        }
        -----------------------------------------------------------------------------------------------------

----Button Designer--------

 <asp:Button ID="btnGenerateReport" runat="server" OnClick="btnGenerateReport_Click" OnClientClick="document.forms[0].target='_blank';"
  Text="Report" />



        Some More Code for PDF Print..

        ---------------------------------------------------------------------------------------------------------

        using System;
using System.Data;
using System.Data.SqlClient;
using System.IO;
using System.Text;
using System.Web.UI;
using iTextSharp.text;
using iTextSharp.text.pdf;

namespace Report
{
    public partial class Contact : Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {

        }

        protected void btnGenerateReport_Click(object sender, EventArgs e)
        {
            //try
            //{
            //    // Get data from database
            //    DataTable dt = GetScurptData();

            //    if (dt != null && dt.Rows.Count > 0)
            //    {
            //        // Generate HTML report
            //        string htmlReport = GenerateHtmlReport(dt);

            //        // Send HTML as response (opens in new tab if target="_blank")
            //        Response.Clear();
            //        Response.ContentType = "text/html";
            //        Response.Write(htmlReport);
            //        Response.Flush();
            //        Response.End();
            //    }
            //    else
            //    {
            //        ClientScript.RegisterStartupScript(this.GetType(), "alert",
            //            "alert('No data found.');", true);
            //    }
            //}
            //catch (Exception ex)
            //{
            //    ClientScript.RegisterStartupScript(this.GetType(), "alert",
            //        "alert('Error: " + ex.Message.Replace("'", "\\'") + "');", true);
            //}

            try
            {
                // Get data from database
                DataTable dt = GetScurptData();

                if (dt != null && dt.Rows.Count > 0)
                {
                    // Generate PDF report
                    byte[] pdfBytes = GeneratePdfReport(dt);

                    // Send PDF to browser in new tab
                    Response.Clear();
                    Response.ContentType = "application/pdf";
                    Response.AddHeader("Content-Disposition", "inline; filename=WaterQualityReport.pdf");
                    Response.BinaryWrite(pdfBytes);
                    Response.Flush();
                    Response.End();
                }
                else
                {
                    ClientScript.RegisterStartupScript(this.GetType(), "alert",
                        "alert('No data found.');", true);
                }
            }
            catch (Exception ex)
            {
                ClientScript.RegisterStartupScript(this.GetType(), "alert",
                    "alert('Error: " + ex.Message.Replace("'", "\\'") + "');", true);
            }
        }

        private DataTable GetScurptData()
        {
            DataTable dt = new DataTable();
            dt.Columns.Add("Type", typeof(string));
            dt.Columns.Add("Site", typeof(string));
            dt.Columns.Add("Message", typeof(string));

            // OPTION 1: Using dummy data for testing
            dt.Rows.Add("A", "", "Added:75 Updated:77 Warnings:405 Outliers:227 FROM:2022-10-28 09:57:44");
            dt.Rows.Add("", "", "TO:2022-11-23 17:09:50");
            dt.Rows.Add("O", "SC000", "D:20221018 T:0827 d:00.1 95th%:SPEC_COND 38 Sample Value: 62");
            dt.Rows.Add("", "SC000", "D:20221114 T:1139 d:00.1 95th%:TDS 16.776 Sample Value: 46");
            dt.Rows.Add("", "SC000", "D:20221024 T:0932 d:00.1 95th%:PHOSPHU .035 Sample Value: 0.10");
            dt.Rows.Add("", "SC105", "D:20221011 T:0805 d:00.1 95th%:MAGNESIUM 12.993 Sample Value: 13");
            dt.Rows.Add("", "SC105", "D:20221011 T:0805 d:00.1 95th%:POTASSIUM .702 Sample Value: 0.72");
            dt.Rows.Add("", "SC201", "D:20221031 T:1103 d:00.1 95th%:CHLORIDE 16 Sample Value: 20");
            dt.Rows.Add("", "SC201", "D:20221031 T:1108 d:00.1 95th%:CHLORIDE 16 Sample Value: 20");
            dt.Rows.Add("", "SC203", "D:20221018 T:0916 d:00.1 95th%:CHLORIDE 142 Sample Value: 150");
            dt.Rows.Add("", "SC204", "D:20221018 T:1008 d:00.1 95th%:SULFATE 162.371 Sample Value: 200");
            dt.Rows.Add("", "SC205", "D:20221018 T:1101 d:00.1 95th%:CHLORIDE 54.48 Sample Value: 56");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:MAGNESIUM 14 Sample Value: 19");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:BROMIDE .51 Sample Value: 0.78");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:AMMONIA .632 Sample Value: 2.1");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:NITRATE 1.2 Sample Value: 1.7");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:POTTASIUM 9.698 Sample Value: 10");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 OLD_MAX:KJELDAHL 1.95 Sample Value: 2.7");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:TOTHARD 249.3 Sample Value: 250");
            dt.Rows.Add("O", "SC000", "D:20221018 T:0827 d:00.1 95th%:SPEC_COND 38 Sample Value: 62");
            dt.Rows.Add("", "SC000", "D:20221114 T:1139 d:00.1 95th%:TDS 16.776 Sample Value: 46");
            dt.Rows.Add("", "SC000", "D:20221024 T:0932 d:00.1 95th%:PHOSPHU .035 Sample Value: 0.10");
            dt.Rows.Add("", "SC105", "D:20221011 T:0805 d:00.1 95th%:MAGNESIUM 12.993 Sample Value: 13");
            dt.Rows.Add("", "SC105", "D:20221011 T:0805 d:00.1 95th%:POTASSIUM .702 Sample Value: 0.72");
            dt.Rows.Add("", "SC201", "D:20221031 T:1103 d:00.1 95th%:CHLORIDE 16 Sample Value: 20");
            dt.Rows.Add("", "SC201", "D:20221031 T:1108 d:00.1 95th%:CHLORIDE 16 Sample Value: 20");
            dt.Rows.Add("", "SC203", "D:20221018 T:0916 d:00.1 95th%:CHLORIDE 142 Sample Value: 150");
            dt.Rows.Add("", "SC204", "D:20221018 T:1008 d:00.1 95th%:SULFATE 162.371 Sample Value: 200");
            dt.Rows.Add("", "SC205", "D:20221018 T:1101 d:00.1 95th%:CHLORIDE 54.48 Sample Value: 56");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:MAGNESIUM 14 Sample Value: 19");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:BROMIDE .51 Sample Value: 0.78");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:AMMONIA .632 Sample Value: 2.1");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:NITRATE 1.2 Sample Value: 1.7");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:POTTASIUM 9.698 Sample Value: 10");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 OLD_MAX:KJELDAHL 1.95 Sample Value: 2.7");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:TOTHARD 249.3 Sample Value: 250");
            dt.Rows.Add("O", "SC000", "D:20221018 T:0827 d:00.1 95th%:SPEC_COND 38 Sample Value: 62");
            dt.Rows.Add("", "SC000", "D:20221114 T:1139 d:00.1 95th%:TDS 16.776 Sample Value: 46");
            dt.Rows.Add("", "SC000", "D:20221024 T:0932 d:00.1 95th%:PHOSPHU .035 Sample Value: 0.10");
            dt.Rows.Add("", "SC105", "D:20221011 T:0805 d:00.1 95th%:MAGNESIUM 12.993 Sample Value: 13");
            dt.Rows.Add("", "SC105", "D:20221011 T:0805 d:00.1 95th%:POTASSIUM .702 Sample Value: 0.72");
            dt.Rows.Add("", "SC201", "D:20221031 T:1103 d:00.1 95th%:CHLORIDE 16 Sample Value: 20");
            dt.Rows.Add("", "SC201", "D:20221031 T:1108 d:00.1 95th%:CHLORIDE 16 Sample Value: 20");
            dt.Rows.Add("", "SC203", "D:20221018 T:0916 d:00.1 95th%:CHLORIDE 142 Sample Value: 150");
            dt.Rows.Add("", "SC204", "D:20221018 T:1008 d:00.1 95th%:SULFATE 162.371 Sample Value: 200");
            dt.Rows.Add("", "SC205", "D:20221018 T:1101 d:00.1 95th%:CHLORIDE 54.48 Sample Value: 56");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:MAGNESIUM 14 Sample Value: 19");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:BROMIDE .51 Sample Value: 0.78");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:AMMONIA .632 Sample Value: 2.1");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:NITRATE 1.2 Sample Value: 1.7");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:POTTASIUM 9.698 Sample Value: 10");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 OLD_MAX:KJELDAHL 1.95 Sample Value: 2.7");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:TOTHARD 249.3 Sample Value: 250");
            dt.Rows.Add("O", "SC000", "D:20221018 T:0827 d:00.1 95th%:SPEC_COND 38 Sample Value: 62");
            dt.Rows.Add("", "SC000", "D:20221114 T:1139 d:00.1 95th%:TDS 16.776 Sample Value: 46");
            dt.Rows.Add("", "SC000", "D:20221024 T:0932 d:00.1 95th%:PHOSPHU .035 Sample Value: 0.10");
            dt.Rows.Add("", "SC105", "D:20221011 T:0805 d:00.1 95th%:MAGNESIUM 12.993 Sample Value: 13");
            dt.Rows.Add("", "SC105", "D:20221011 T:0805 d:00.1 95th%:POTASSIUM .702 Sample Value: 0.72");
            dt.Rows.Add("", "SC201", "D:20221031 T:1103 d:00.1 95th%:CHLORIDE 16 Sample Value: 20");
            dt.Rows.Add("", "SC201", "D:20221031 T:1108 d:00.1 95th%:CHLORIDE 16 Sample Value: 20");
            dt.Rows.Add("", "SC203", "D:20221018 T:0916 d:00.1 95th%:CHLORIDE 142 Sample Value: 150");
            dt.Rows.Add("", "SC204", "D:20221018 T:1008 d:00.1 95th%:SULFATE 162.371 Sample Value: 200");
            dt.Rows.Add("", "SC205", "D:20221018 T:1101 d:00.1 95th%:CHLORIDE 54.48 Sample Value: 56");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:MAGNESIUM 14 Sample Value: 19");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:BROMIDE .51 Sample Value: 0.78");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:AMMONIA .632 Sample Value: 2.1");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:NITRATE 1.2 Sample Value: 1.7");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:POTTASIUM 9.698 Sample Value: 10");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 OLD_MAX:KJELDAHL 1.95 Sample Value: 2.7");
            dt.Rows.Add("", "SC215", "D:20221010 T:1440 d:00.1 95th%:TOTHARD 249.3 Sample Value: 250");

            return dt;

            // OPTION 2: Uncomment below to use actual database connection
            /*
            string connectionString = System.Configuration.ConfigurationManager.ConnectionStrings["YourConnectionString"].ConnectionString;
            
            string query = @"SELECT typeof [Type], site_name [Site], xmsg [Message] 
                            FROM befs_admin.scurpt
                            WHERE dateof = @dateof
                            ORDER BY typeof, site_name";

            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                connection.Open();
                using (SqlCommand command = new SqlCommand(query, connection))
                {
                    // Replace with your session variable or parameter
                    command.Parameters.AddWithValue("@dateof", "2022-11-23");
                    
                    using (SqlDataAdapter da = new SqlDataAdapter(command))
                    {
                        da.Fill(dt);
                    }
                }
            }
            return dt;
            */
        }

        private byte[] GeneratePdfReport(DataTable data)
        {
            using (MemoryStream ms = new MemoryStream())
            {
                // Increase top margin to make space for header on every page
                Document document = new Document(PageSize.A4, 36, 36, 60, 36);

                PdfWriter writer = PdfWriter.GetInstance(document, ms);

                // ===========================
                // INLINE PAGE HEADER EVENT
                // ===========================

                var dynamicHedaerValue = "Alok Aswal";
                writer.PageEvent = new PdfPageEventHelperInline(dynamicHedaerValue);

                document.Open();

                // Create fonts
                BaseFont bf = BaseFont.CreateFont(BaseFont.COURIER, BaseFont.CP1252, BaseFont.NOT_EMBEDDED);
                Font normalFont = new Font(bf, 10, Font.NORMAL);
                Font underlineFont = new Font(bf, 10, Font.UNDERLINE);

                // Create table with 3 columns
                PdfPTable table = new PdfPTable(3);
                table.WidthPercentage = 100;
                table.SetWidths(new float[] { 0.8f, 1.0f, 6.0f });

                // Header row
                PdfPCell headerCell;

                headerCell = new PdfPCell(new Phrase("Type", normalFont));
                headerCell.Border = Rectangle.NO_BORDER;
                table.AddCell(headerCell);

                headerCell = new PdfPCell(new Phrase("Site", normalFont));
                headerCell.Border = Rectangle.NO_BORDER;
                table.AddCell(headerCell);

                headerCell = new PdfPCell(new Phrase("Message", normalFont));
                headerCell.Border = Rectangle.NO_BORDER;
                table.AddCell(headerCell);

                // Data rows
                foreach (DataRow row in data.Rows)
                {
                    PdfPCell cell;

                    cell = new PdfPCell(new Phrase(row["Type"].ToString(), normalFont));
                    cell.Border = Rectangle.NO_BORDER;
                    table.AddCell(cell);

                    cell = new PdfPCell(new Phrase(row["Site"].ToString(), normalFont));
                    cell.Border = Rectangle.NO_BORDER;
                    table.AddCell(cell);

                    cell = new PdfPCell(new Phrase(row["Message"].ToString(), underlineFont));
                    cell.Border = Rectangle.NO_BORDER;
                    table.AddCell(cell);
                }

                document.Add(table);
                document.Close();
                writer.Close();

                return ms.ToArray();
            }
        }

        //private byte[] GeneratePdfReport(DataTable data)
        //{
        //    using (MemoryStream ms = new MemoryStream())
        //    {
        //        // Create PDF document
        //        Document document = new Document(PageSize.A4, 36, 36, 36, 36);
        //        PdfWriter writer = PdfWriter.GetInstance(document, ms);
        //        document.Open();

        //        // Create fonts
        //        BaseFont bf = BaseFont.CreateFont(BaseFont.COURIER, BaseFont.CP1252, BaseFont.NOT_EMBEDDED);
        //        Font normalFont = new Font(bf, 10, Font.NORMAL);
        //        Font underlineFont = new Font(bf, 10, Font.UNDERLINE);

        //        // Create table with 3 columns (no borders)
        //        PdfPTable table = new PdfPTable(3);
        //        table.WidthPercentage = 100;
        //        table.SetWidths(new float[] { 0.8f, 1.0f, 6.0f });

        //        // Add header cells (no border, no underline, no bold)
        //        PdfPCell headerCell;

        //        headerCell = new PdfPCell(new Phrase("Type", normalFont));
        //        headerCell.Border = Rectangle.NO_BORDER;
        //        headerCell.Padding = 4;
        //        table.AddCell(headerCell);

        //        headerCell = new PdfPCell(new Phrase("Site", normalFont));
        //        headerCell.Border = Rectangle.NO_BORDER;
        //        headerCell.Padding = 4;
        //        table.AddCell(headerCell);

        //        headerCell = new PdfPCell(new Phrase("Message", normalFont));
        //        headerCell.Border = Rectangle.NO_BORDER;
        //        headerCell.Padding = 4;
        //        headerCell.PaddingBottom = 10;
        //        table.AddCell(headerCell);

        //        // Add data rows (no borders, message underlined)
        //        foreach (DataRow row in data.Rows)
        //        {
        //            PdfPCell cell;

        //            // Type column
        //            cell = new PdfPCell(new Phrase(row["Type"].ToString(), normalFont));
        //            cell.Border = Rectangle.NO_BORDER;
        //            cell.Padding = 4;
        //            cell.VerticalAlignment = Element.ALIGN_TOP;
        //            table.AddCell(cell);

        //            // Site column
        //            cell = new PdfPCell(new Phrase(row["Site"].ToString(), normalFont));
        //            cell.Border = Rectangle.NO_BORDER;
        //            cell.Padding = 4;
        //            cell.VerticalAlignment = Element.ALIGN_TOP;
        //            table.AddCell(cell);

        //            // Message column (underlined)
        //            cell = new PdfPCell(new Phrase(row["Message"].ToString(), underlineFont));
        //            cell.Border = Rectangle.NO_BORDER;
        //            cell.Padding = 4;
        //            cell.VerticalAlignment = Element.ALIGN_TOP;
        //            table.AddCell(cell);
        //        }

        //        document.Add(table);
        //        document.Close();
        //        writer.Close();

        //        return ms.ToArray();
        //    }
        //}

        //private byte[] GeneratePdfReport(DataTable data)
        //{
        //    using (MemoryStream ms = new MemoryStream())
        //    {
        //        // Create PDF document
        //        Document document = new Document(PageSize.A4, 36, 36, 36, 36);
        //        PdfWriter writer = PdfWriter.GetInstance(document, ms);
        //        document.Open();

        //        // Create fonts
        //        BaseFont bf = BaseFont.CreateFont(BaseFont.COURIER, BaseFont.CP1252, BaseFont.NOT_EMBEDDED);
        //        Font headerFont = new Font(bf, 11, Font.BOLD);
        //        Font cellFont = new Font(bf, 10, Font.NORMAL);
        //        Font messageFont = new Font(bf, 9, Font.NORMAL);

        //        // Create table with 3 columns
        //        PdfPTable table = new PdfPTable(3);
        //        table.WidthPercentage = 100;
        //        table.SetWidths(new float[] { 0.6f, 0.8f, 6.1f });

        //        // Add header cells
        //        PdfPCell headerCell;

        //        headerCell = new PdfPCell(new Phrase("Type", headerFont));
        //        headerCell.Border = Rectangle.BOX;
        //        headerCell.BorderWidth = 1f;
        //        headerCell.BorderColor = BaseColor.BLACK;
        //        headerCell.Padding = 4;
        //        headerCell.BackgroundColor = BaseColor.WHITE;
        //        table.AddCell(headerCell);

        //        headerCell = new PdfPCell(new Phrase("Site", headerFont));
        //        headerCell.Border = Rectangle.BOX;
        //        headerCell.BorderWidth = 1f;
        //        headerCell.BorderColor = BaseColor.BLACK;
        //        headerCell.Padding = 4;
        //        headerCell.BackgroundColor = BaseColor.WHITE;
        //        table.AddCell(headerCell);

        //        headerCell = new PdfPCell(new Phrase("Message", headerFont));
        //        headerCell.Border = Rectangle.BOX;
        //        headerCell.BorderWidth = 1f;
        //        headerCell.BorderColor = BaseColor.BLACK;
        //        headerCell.Padding = 4;
        //        headerCell.BackgroundColor = BaseColor.WHITE;
        //        table.AddCell(headerCell);

        //        // Add data rows
        //        foreach (DataRow row in data.Rows)
        //        {
        //            PdfPCell cell;

        //            cell = new PdfPCell(new Phrase(row["Type"].ToString(), cellFont));
        //            cell.Border = Rectangle.BOX;
        //            cell.BorderWidth = 1f;
        //            cell.BorderColor = BaseColor.BLACK;
        //            cell.Padding = 4;
        //            cell.BackgroundColor = BaseColor.WHITE;
        //            table.AddCell(cell);

        //            cell = new PdfPCell(new Phrase(row["Site"].ToString(), cellFont));
        //            cell.Border = Rectangle.BOX;
        //            cell.BorderWidth = 1f;
        //            cell.BorderColor = BaseColor.BLACK;
        //            cell.Padding = 4;
        //            cell.BackgroundColor = BaseColor.WHITE;
        //            table.AddCell(cell);

        //            cell = new PdfPCell(new Phrase(row["Message"].ToString(), messageFont));
        //            cell.Border = Rectangle.BOX;
        //            cell.BorderWidth = 1f;
        //            cell.BorderColor = BaseColor.BLACK;
        //            cell.Padding = 4;
        //            cell.BackgroundColor = BaseColor.WHITE;
        //            table.AddCell(cell);
        //        }

        //        document.Add(table);
        //        document.Close();
        //        writer.Close();

        //        return ms.ToArray();
        //    }
        //}

        private string GenerateHtmlReport(DataTable data)
        {
            StringBuilder html = new StringBuilder();

            html.Append("<!DOCTYPE html>");
            html.Append("<html>");
            html.Append("<head>");
            html.Append("<title>Water Quality Report</title>");
            html.Append("<style>");
            html.Append("@media print { body { margin: 0; } }");
            html.Append("body { font-family: 'Courier New', monospace; margin: 20px; background: white; }");
            html.Append("table { width: 100%; border-collapse: collapse; border: 1px solid #000; }");
            html.Append("th { background-color: #fff; color: #000; padding: 6px 8px; text-align: left; ");
            html.Append("border: 1px solid #000; font-weight: bold; font-size: 11pt; }");
            html.Append("td { border: 1px solid #000; padding: 4px 8px; vertical-align: top; font-size: 10pt; }");
            html.Append(".col-message { font-size: 9pt; }");
            html.Append("</style>");
            html.Append("</head>");
            html.Append("<body>");
            html.Append("<table>");
            html.Append("<thead>");
            html.Append("<tr>");
            html.Append("<th style='width: 60px;'>Type</th>");
            html.Append("<th style='width: 80px;'>Site</th>");
            html.Append("<th>Message</th>");
            html.Append("</tr>");
            html.Append("</thead>");
            html.Append("<tbody>");

            foreach (DataRow row in data.Rows)
            {
                html.Append("<tr>");
                html.Append("<td>" + Server.HtmlEncode(row["Type"].ToString()) + "</td>");
                html.Append("<td>" + Server.HtmlEncode(row["Site"].ToString()) + "</td>");
                html.Append("<td class='col-message'>" + Server.HtmlEncode(row["Message"].ToString()) + "</td>");
                html.Append("</tr>");
            }

            html.Append("</tbody>");
            html.Append("</table>");
            html.Append("</body>");
            html.Append("</html>");

            return html.ToString();
        }
    }

    public class PdfPageEventHelperInline : PdfPageEventHelper
    {
        private string _headerText;
        public PdfPageEventHelperInline(string headerText)
        {
            _headerText = headerText;
        }
        public override void OnEndPage(PdfWriter writer, Document document)
        {
            PdfPTable header = new PdfPTable(1);
            header.TotalWidth = document.PageSize.Width - document.LeftMargin - document.RightMargin;

            PdfPCell cell = new PdfPCell(new Phrase(_headerText,
                new Font(Font.FontFamily.COURIER, 12, Font.BOLD)))
            {
                Border = Rectangle.NO_BORDER,
                HorizontalAlignment = Element.ALIGN_CENTER,
                PaddingBottom = 8
            };

            header.AddCell(cell);

            header.WriteSelectedRows(
                0, -1,
                document.LeftMargin,
                document.PageSize.Height - 20,
                writer.DirectContent
            );
        }
    }

}

----------------------------------------------------------------------------------

public class PdfPageEventHelperInline : PdfPageEventHelper
{
    private string _runDateTime;
    private PdfTemplate _totalPagesTemplate;
    private BaseFont _bf;

    public PdfPageEventHelperInline(string runDateTime)
    {
        _runDateTime = runDateTime;
    }

    public override void OnOpenDocument(PdfWriter writer, Document document)
    {
        _bf = BaseFont.CreateFont(BaseFont.COURIER, BaseFont.CP1252, BaseFont.NOT_EMBEDDED);
        _totalPagesTemplate = writer.DirectContent.CreateTemplate(50, 12);
    }

    public override void OnEndPage(PdfWriter writer, Document document)
    {
        PdfContentByte cb = writer.DirectContent;

        cb.BeginText();
        cb.SetFontAndSize(_bf, 10);

        // LEFT HEADER: Date/Time
        cb.ShowTextAligned(
            Element.ALIGN_LEFT,
            $"Date/Time Sample Update Ran: {_runDateTime}",
            document.LeftMargin,
            document.PageSize.Height - 30,
            0);

        // RIGHT HEADER: Page X of
        cb.ShowTextAligned(
            Element.ALIGN_RIGHT,
            $"Page: {writer.PageNumber} of",
            document.PageSize.Width - document.RightMargin - 50,
            document.PageSize.Height - 30,
            0);

        cb.EndText();

        // Placeholder for total pages
        cb.AddTemplate(
            _totalPagesTemplate,
            document.PageSize.Width - document.RightMargin,
            document.PageSize.Height - 30);
    }

    public override void OnCloseDocument(PdfWriter writer, Document document)
    {
        _totalPagesTemplate.BeginText();
        _totalPagesTemplate.SetFontAndSize(_bf, 10);
        _totalPagesTemplate.ShowText(writer.PageNumber.ToString());
        _totalPagesTemplate.EndText();
    }
}


-----------------------------------------------------------------------------------------------------------------------------------------

public class PdfPageEventHelperInline : PdfPageEventHelper
{
    private string _runDateTime;
    private PdfTemplate _totalPagesTemplate;
    private BaseFont _bf;

    public PdfPageEventHelperInline(string runDateTime)
    {
        _runDateTime = runDateTime;
    }

    public override void OnOpenDocument(PdfWriter writer, Document document)
    {
        _bf = BaseFont.CreateFont(BaseFont.COURIER, BaseFont.CP1252, BaseFont.NOT_EMBEDDED);
        _totalPagesTemplate = writer.DirectContent.CreateTemplate(50, 12);
    }

    public override void OnEndPage(PdfWriter writer, Document document)
    {
        PdfContentByte cb = writer.DirectContent;

        // ----- DATE/TIME BOX -----
        PdfPTable headerTable = new PdfPTable(1);
        headerTable.TotalWidth = 260;

        PdfPCell dateCell = new PdfPCell(
            new Phrase($"Date/Time Sample Update Ran: {_runDateTime}",
            new Font(_bf, 10)))
        {
            Border = Rectangle.BOX,
            Padding = 6
        };

        headerTable.AddCell(dateCell);

        headerTable.WriteSelectedRows(
            0, -1,
            document.LeftMargin,
            document.PageSize.Height - 25,
            cb);

        // ----- PAGE NUMBER -----
        cb.BeginText();
        cb.SetFontAndSize(_bf, 10);

        cb.ShowTextAligned(
            Element.ALIGN_RIGHT,
            $"Page: {writer.PageNumber} of",
            document.PageSize.Width - document.RightMargin - 50,
            document.PageSize.Height - 35,
            0);

        cb.EndText();

        cb.AddTemplate(
            _totalPagesTemplate,
            document.PageSize.Width - document.RightMargin,
            document.PageSize.Height - 35);
    }

    public override void OnCloseDocument(PdfWriter writer, Document document)
    {
        _totalPagesTemplate.BeginText();
        _totalPagesTemplate.SetFontAndSize(_bf, 10);
        _totalPagesTemplate.ShowText(writer.PageNumber.ToString());
        _totalPagesTemplate.EndText();
    }
}
-----------------------------

public override void OnEndPage(PdfWriter writer, Document document)
{
    PdfContentByte cb = writer.DirectContent;

    Font font = new Font(_bf, 10);

    float yPos = document.PageSize.Height - 35;

    // -------- LABEL TEXT (outside box) --------
    ColumnText.ShowTextAligned(
        cb,
        Element.ALIGN_LEFT,
        new Phrase("Date/Time Sample Update Ran:", font),
        document.LeftMargin,
        yPos,
        0);

    // -------- DATE VALUE INSIDE BOX --------
    PdfPTable dateTable = new PdfPTable(1);
    dateTable.TotalWidth = 120;

    PdfPCell dateCell = new PdfPCell(
        new Phrase(_runDateTime, font))
    {
        Border = Rectangle.BOX,
        Padding = 4,
        HorizontalAlignment = Element.ALIGN_CENTER
    };

    dateTable.AddCell(dateCell);

    dateTable.WriteSelectedRows(
        0, -1,
        document.LeftMargin + 190,   // Adjust spacing after label
        yPos + 12,
        cb);

    // -------- PAGE NUMBER (RIGHT SIDE) --------
    cb.BeginText();
    cb.SetFontAndSize(_bf, 10);

    cb.ShowTextAligned(
        Element.ALIGN_RIGHT,
        $"Page: {writer.PageNumber} of",
        document.PageSize.Width - document.RightMargin - 50,
        yPos,
        0);

    cb.EndText();

    cb.AddTemplate(
        _totalPagesTemplate,
        document.PageSize.Width - document.RightMargin,
        yPos);
}

----------------------

private byte[] GeneratePdfReport(DataTable data)
{
    using (MemoryStream ms = new MemoryStream())
    {
        Document document = new Document(PageSize.A4, 36, 36, 80, 36);
        PdfWriter writer = PdfWriter.GetInstance(document, ms);

        string runDateTime = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
        writer.PageEvent = new PdfPageEventHelperInline(runDateTime);

        document.Open();

        BaseFont bf = BaseFont.CreateFont(BaseFont.COURIER, BaseFont.CP1252, BaseFont.NOT_EMBEDDED);
        Font normalFont = new Font(bf, 10, Font.NORMAL);
        Font underlineFont = new Font(bf, 10, Font.UNDERLINE);

        PdfPTable table = new PdfPTable(3);
        table.WidthPercentage = 100;
        table.SetWidths(new float[] { 0.8f, 1.0f, 6.0f });

        // Column headers
        table.AddCell(new PdfPCell(new Phrase("Type", normalFont)) { Border = Rectangle.NO_BORDER });
        table.AddCell(new PdfPCell(new Phrase("Site", normalFont)) { Border = Rectangle.NO_BORDER });
        table.AddCell(new PdfPCell(new Phrase("Message", normalFont)) { Border = Rectangle.NO_BORDER });

        string previousType = null;

        foreach (DataRow row in data.Rows)
        {
            PdfPCell cell;

            // ---- TYPE (Suppress repetition) ----
            string currentType = row["Type"].ToString();
            string displayType = currentType == previousType ? string.Empty : currentType;

            cell = new PdfPCell(new Phrase(displayType, normalFont))
            {
                Border = Rectangle.NO_BORDER,
                VerticalAlignment = Element.ALIGN_TOP
            };
            table.AddCell(cell);

            previousType = currentType;

            // ---- SITE ----
            cell = new PdfPCell(new Phrase(row["Site"].ToString(), normalFont))
            {
                Border = Rectangle.NO_BORDER,
                VerticalAlignment = Element.ALIGN_TOP
            };
            table.AddCell(cell);

            // ---- MESSAGE ----
            cell = new PdfPCell(new Phrase(row["Message"].ToString(), underlineFont))
            {
                Border = Rectangle.NO_BORDER,
                VerticalAlignment = Element.ALIGN_TOP
            };
            table.AddCell(cell);
        }

        document.Add(table);
        document.Close();
        writer.Close();

        return ms.ToArray();
    }
}



        
    
    
