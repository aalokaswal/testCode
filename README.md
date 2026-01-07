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
    
    
