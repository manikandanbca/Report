// Essbase Report Script: Cross Join Columns Example
// Cross-joining Scenario and Time dimensions

{ } // Report Script Block Start

// --- Page Dimensions (Filters) ---
<PAGE (Year, Market)
  "FY2025"     // Example: Filter for Fiscal Year 2025
  "California" // Example: Filter for California market

// --- Column Dimensions (The Cross Join) ---
// Specify the dimensions to be cross-joined in the columns
<COLUMN (Scenario, Time) // Dimension order matters for nesting

  // Members for the first dimension (Scenario) - Outer loop
  Actual Budget

  // Members for the second dimension (Time) - Inner loop
  Jan Feb

  // How Essbase processes this:
  // 1. Takes "Actual" (from Scenario) -> Pairs with "Jan" (from Time) -> Column 1: Actual / Jan
  // 2. Takes "Actual" (from Scenario) -> Pairs with "Feb" (from Time) -> Column 2: Actual / Feb
  // 3. Takes "Budget" (from Scenario) -> Pairs with "Jan" (from Time) -> Column 3: Budget / Jan
  // 4. Takes "Budget" (from Scenario) -> Pairs with "Feb" (from Time) -> Column 4: Budget / Feb

// --- Row Dimensions ---
<ROW (Measures) // Dimension for the rows
  Sales
  COGS
  "Gross Margin" // Example member name with space

// --- Formatting (Optional) ---
<DECIMAL 2      // Show 2 decimal places for data
<WIDTH 14       // Set column width
<SYM            // Apply width/decimal symmetrically
<SUPEMPTYROWS   // Suppress rows if all data is #Missing or zero

! // Execute the report generation
