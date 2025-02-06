a!localVariables(
  
  /* Stores the selected party for drill-down */
  local!selectedParty: null,

  /* Dropdown for manual selection (optional) */
  a!sectionLayout(
    contents: {
      if(
        isnull(local!selectedParty),
        a!richTextDisplayField(
          value: a!richTextItem(
            text: "Click on a party from the grid below to view its margin distribution.",
            color: "SECONDARY"
          )
        ),
        a!buttonLayout(
          primaryButtons: a!buttonWidget(
            label: "Back to Vote % by Party",
            value: null,
            saveInto: a!save(local!selectedParty, null),
            style: "SECONDARY"
          )
        )
      ),

      /* Pie Chart Field */
      a!pieChartField(
        label: if(
          isnull(local!selectedParty),
          "Vote %age By Party",
          "Margin Distribution for " & local!selectedParty
        ),
        labelPosition: "ABOVE",
        
        /* Conditional Data Based on Drill-Down */
        data: a!recordData(
          recordType: cons!DR_ELECTION_RESULT,
          filters: if(
            isnull(local!selectedParty),
            {}, /* No filter at first level */
            a!queryFilter(
              field: cons!DR_ELECTION_RESULT.LEADING_PARTY,
              operator: "=",
              value: local!selectedParty
            )
          )
        ),

        /* Pie Chart Configuration */
        config: a!pieChartConfig(
          primaryGrouping: a!grouping(
            field: if(
              isnull(local!selectedParty),
              cons!DR_ELECTION_RESULT.LEADING_PARTY, /* First level: Party */
              cons!DR_ELECTION_RESULT.CONSTITUENCY /* Second level: Constituency */
            ),
            alias: "group"
          ),
          measures: a!measure(
            field: cons!DR_ELECTION_RESULT.MARGIN,
            alias: "margin",
            function: "SUM"
          )
        ),

        showAsPercentage: true,
        showTooltips: true,
        colorScheme: "SUNSET",
        style: "PIE",
        seriesLabelStyle: "ON_CHART",
        height: "MEDIUM"
      ),

      /* Grid to Select a Party for Drill-Down */
      if(
        isnull(local!selectedParty),
        a!gridField(
          label: "Select a Party",
          totalCount: -1,
          data: a!recordData(
            recordType: cons!DR_ELECTION_RESULT,
            config: a!gridConfig(
              columns: {
                a!gridColumn(
                  label: "Leading Party",
                  value: fv!row.LEADING_PARTY
                ),
                a!gridColumn(
                  label: "Seats Won",
                  value: count(
                    where(
                      local!electionData.data.LEADING_PARTY,
                      fv!row.LEADING_PARTY
                    )
                  )
                ),
                a!gridColumn(
                  label: "View Margin",
                  value: a!richTextDisplayField(
                    value: a!richTextItem(
                      text: "View",
                      link: a!dynamicLink(
                        value: fv!row.LEADING_PARTY,
                        saveInto: a!save(local!selectedParty, fv!row.LEADING_PARTY)
                      ),
                      color: "ACCENT"
                    )
                  )
                )
              }
            )
          )
        ),
        {}
      )
    }
  )
)
