@Data
@NoArgsConstructor
@AllArgsConstructor
public class GlobalFilterRequest {
    private List<String> useCaseNo;
    private List<String> useCaseOwnership;
    private List<String> useCaseOwner;
    private List<String> useCaseAccountableExecutive;
    private List<String> useCaseCategory;
    private List<String> projectOverallRagStatus;

    private List<String> smtAccountableExecutive;
    private List<String> milestoneOwner;
    private List<String> deliverableOwner;

    private List<String> programName;
    private List<String> initiativeName;
    private List<String> workEffortName;
    private List<String> derivedProgramCategory;

    private List<String> occConsentOrderArticle;
    private List<String> subportfolioName;

    private List<Integer> useCaseDueYear;
    private List<String> useCaseDueQuarter;
    private List<String> useCaseDueMonth;

    private List<Integer> milestoneYear;
    private List<String> milestoneQuarter;
    private List<String> milestoneMonth;

    private List<Integer> deliverableYear;
    private List<String> deliverableQuarter;
    private List<String> deliverableMonth;

    private List<String> milestoneRagStatus;
    private List<String> deliverableRagStatus;
}











// global filter 


package com.yourcompany.pts.util;

import com.yourcompany.pts.dto.GlobalFilterRequest;

import java.util.List;
import java.util.StringJoiner;

public class GlobalFilterBuilder {

    private GlobalFilterBuilder() {
    }

    public static String build(GlobalFilterRequest filter) {

        if (filter == null) {
            return "";
        }

        StringJoiner where = new StringJoiner("\nAND ", "\nAND ", "");

        /* ================= Use Case Filters ================= */
        addIn(where, "MS.USE_CASE_NO", filter.getUseCaseNo());
        addIn(where, "MS.USE_CASE_OWNERSHIP", filter.getUseCaseOwnership());
        addIn(where, "MS.USE_CASE_OWNER", filter.getUseCaseOwner());
        addIn(where, "MS.USE_CASE_ACCOUNTABLE_EXECUTIVE", filter.getUseCaseAccountableExecutive());
        addIn(where, "MS.USE_CASE_CATEGORY", filter.getUseCaseCategory());
        addIn(where, "MS.PROJECT_OVERALL_RAG_STATUS", filter.getProjectOverallRagStatus());

        /* ================= Ownership ================= */
        addIn(where, "MS.SMT_ACCOUNTABLE_EXECUTIVE", filter.getSmtAccountableExecutive());
        addIn(where, "MS.MILESTONE_OWNER", filter.getMilestoneOwner());
        addIn(where, "DEL.DELIVERABLE_OWNER", filter.getDeliverableOwner());

        /* ================= Program / Work ================= */
        addIn(where, "MS.PROGRAM_NAME", filter.getProgramName());
        addIn(where, "DEL.INITIATIVE_NAME", filter.getInitiativeName());
        addIn(where, "DEL.WORK_EFFORT_NAME", filter.getWorkEffortName());
        addIn(where, "DEL.DERIVED_PROGRAM_CATEGORY", filter.getDerivedProgramCategory());

        /* ================= Regulatory / Portfolio ================= */
        addIn(where, "MS.OCC_CONSENT_ORDER_ARTICLE", filter.getOccConsentOrderArticle());
        addIn(where, "MS.SUBPORTFOLIO_NAME", filter.getSubportfolioName());

        /* ================= Use Case Due Date ================= */
        addIn(where, "UC.USE_CASE_DUE_DT_YEAR", filter.getUseCaseDueYear());
        addIn(where, "UC.USE_CASE_DUE_DT_QTR", filter.getUseCaseDueQuarter());
        addIn(where, "UC.USE_CASE_DUE_DT_MONTH", filter.getUseCaseDueMonth());

        /* ================= Milestone Due Date ================= */
        addIn(where, "MS.MILESTONE_YEAR", filter.getMilestoneYear());
        addIn(where, "MS.MILESTONE_QTR", filter.getMilestoneQuarter());
        addIn(where, "MS.MILESTONE_MONTH", filter.getMilestoneMonth());

        /* ================= Deliverable Due Date ================= */
        addIn(where, "DEL.DELIVERABLE_YEAR", filter.getDeliverableYear());
        addIn(where, "DEL.DELIVERABLE_QTR", filter.getDeliverableQuarter());
        addIn(where, "DEL.DELIVERABLE_MONTH", filter.getDeliverableMonth());

        /* ================= RAG ================= */
        addIn(where, "MS.MILESTONE_RAG_STATUS", filter.getMilestoneRagStatus());
        addIn(where, "DEL.DELIVERABLE_RAG_STATUS", filter.getDeliverableRagStatus());

        return where.toString();
    }

    /* ================= Utility ================= */
    private static <T> void addIn(StringJoiner where, String column, List<T> values) {
        if (values == null || values.isEmpty()) {
            return;
        }

        String joined = values.stream()
                .map(v -> v instanceof Number ? v.toString() : "'" + escape(v.toString()) + "'")
                .reduce((a, b) -> a + "," + b)
                .orElse("");

        where.add(column + " IN (" + joined + ")");
    }

    private static String escape(String value) {
        return value.replace("'", "''");
    }
}





// filter repo




String sql = baseSql.replace("{{GLOBAL_FILTER}}",
        GlobalFilterBuilder.build(filterRequest));

return namedParameterJdbcTemplate.query(
        sql,
        new BeanPropertyRowMapper<>(UseCaseDashboardDto.class)
);





// dto topcard response 


package com.yourcompany.pts.dto;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class TopCardResponse {
    private String label;
    private Long value;
}


// repo layer 


// common execution 

private Long executeCountQuery(String sql, GlobalFilterRequest filter) {

    String finalSql = sql.replace(
            "{{GLOBAL_FILTER}}",
            GlobalFilterBuilder.build(filter)
    );

    return namedParameterJdbcTemplate.queryForObject(finalSql, Map.of(), Long.class);
}

// topcard 1

public Long getTotalUseCases(GlobalFilterRequest filter) {

    String sql = """
        SELECT COUNT(DISTINCT MS.USE_CASE_NO)
        FROM ( {{BASE_QUERY}} )
        WHERE 1=1
        {{GLOBAL_FILTER}}
        """;

    return executeCountQuery(sql, filter);
}


// Top 2 

public Long getTotalMilestones(GlobalFilterRequest filter) {

    String sql = """
        SELECT COUNT(DISTINCT MS.MILESTONE_ID)
        FROM ( {{BASE_QUERY}} )
        WHERE 1=1
        {{GLOBAL_FILTER}}
        """;

    return executeCountQuery(sql, filter);
}



// Top 3 


public Long getTotalDeliverables(GlobalFilterRequest filter) {

    String sql = """
        SELECT COUNT(DISTINCT DEL.WORK_EFFORT_NAME)
        FROM ( {{BASE_QUERY}} )
        WHERE 1=1
        {{GLOBAL_FILTER}}
        """;

    return executeCountQuery(sql, filter);
}



// service layer 


@Service
@RequiredArgsConstructor
public class TopCardService {

    private final TopCardRepository repository;

    public List<TopCardResponse> getTopCards(GlobalFilterRequest filter) {

        return List.of(
                new TopCardResponse("Total Use Cases",
                        repository.getTotalUseCases(filter)),

                new TopCardResponse("Total Milestones",
                        repository.getTotalMilestones(filter)),

                new TopCardResponse("Total Deliverables",
                        repository.getTotalDeliverables(filter))
        );
    }
}



// controller 


@RestController
@RequestMapping("/api/pts/dashboard")
@RequiredArgsConstructor
public class TopCardController {

    private final TopCardService service;

    @PostMapping("/top-cards")
    public List<TopCardResponse> getTopCards(
            @RequestBody GlobalFilterRequest filter) {

        return service.getTopCards(filter);
    }
}



