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



        
    
    
