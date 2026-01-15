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



        
    
    
