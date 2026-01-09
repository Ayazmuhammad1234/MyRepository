package com.yourcompany.pts.dto;

import lombok.Data;
import java.util.List;

@Data
public class GlobalFilterRequest {

    /* =======================
       1. USE CASE
       ======================= */
    private List<String> useCaseNo;
    private List<String> useCaseOwnership;
    private List<String> useCaseOwner;
    private List<String> useCaseAccountableExecutive;
    private List<String> useCaseRbcmType;          // USE_CASE_CATEGORY
    private List<String> useCaseRagStatus;         // PROJECT_OVERALL_RAG_STATUS

    /* =======================
       2. OWNER
       ======================= */
    private List<String> accountableExecutive;     // SMT_ACCOUNTABLE_EXECUTIVE
    private List<String> milestoneOwner;
    private List<String> deliverableOwner;

    /* =======================
       3. TIMELINE
       ======================= */
    private List<Integer> milestoneYear;
    private List<Integer> deliverableYear;

    /* =======================
       4. WORK EFFORT
       ======================= */
    private List<String> initiative;
    private List<String> program;
    private List<String> project;                  // WORK_EFFORT_NAME
    private List<String> programCategory;           // DERIVED_PROGRAM_CATEGORY

    /* =======================
       5. MILESTONE ATTRIBUTES
       ======================= */
    private List<String> article;                   // OCC_CONSENT_ORDER_ARTICLE
    private List<String> subPortfolio;

    /* =======================
       6. RAG STATUS
       ======================= */
    private List<String> milestoneRag;
    private List<String> deliverableRag;
}







// pts response 



package com.yourcompany.pts.dto;

import lombok.Data;
import java.util.List;
import java.util.Map;

@Data
public class PtsTopCardsResponse {

    // Top Card 1
    private List<Map<String, Object>> ragSummary;

    // Top Card 2
    private List<Map<String, Object>> ownershipCategory;

    // Top Card 3
    private List<Map<String, Object>> yearCategoryRag;
}



// controller 




package com.yourcompany.pts.controller;

import com.yourcompany.pts.dto.GlobalFilterRequest;
import com.yourcompany.pts.dto.PtsTopCardsResponse;
import com.yourcompany.pts.service.PtsTopCardService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/pts/top-cards")
@RequiredArgsConstructor
public class PtsTopCardController {

    private final PtsTopCardService service;

    /**
     * ONE API
     * All global filters applied to ALL top cards
     */
    @PostMapping("/all")
    public PtsTopCardsResponse getAllTopCards(
            @RequestBody GlobalFilterRequest filterRequest) {

        return service.getAllTopCards(filterRequest);
    }
}



//service layer 





package com.yourcompany.pts.service;

import com.yourcompany.pts.dto.GlobalFilterRequest;
import com.yourcompany.pts.dto.PtsTopCardsResponse;

public interface PtsTopCardService {

    PtsTopCardsResponse getAllTopCards(GlobalFilterRequest filterRequest);
}


//




package com.yourcompany.pts.service.impl;

import com.yourcompany.pts.dto.GlobalFilterRequest;
import com.yourcompany.pts.dto.PtsTopCardsResponse;
import com.yourcompany.pts.repository.PtsTopCardRepository;
import com.yourcompany.pts.service.PtsTopCardService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class PtsTopCardServiceImpl implements PtsTopCardService {

    private final PtsTopCardRepository repository;

    @Override
    public PtsTopCardsResponse getAllTopCards(GlobalFilterRequest filterRequest) {

        PtsTopCardsResponse response = new PtsTopCardsResponse();

        // ---------- TOP CARD 1 ----------
        response.setRagSummary(
                repository.getUseCaseCountByOverallRag(filterRequest)
        );

        // ---------- TOP CARD 2 ----------
        response.setOwnershipCategory(
                repository.getUseCaseCountByOwnershipAndCategory(filterRequest)
        );

        // ---------- TOP CARD 3 ----------
        response.setYearCategoryRag(
                repository.getMilestoneCountByYearCategoryRag(filterRequest)
        );

        return response;
    }
}








//repo




package com.yourcompany.pts.repository;

import com.yourcompany.pts.dto.GlobalFilterRequest;
import com.yourcompany.pts.util.GlobalFilterBuilder;
import lombok.RequiredArgsConstructor;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Repository
@RequiredArgsConstructor
public class PtsTopCardRepository {

    private final NamedParameterJdbcTemplate jdbcTemplate;

    private static final String BASE_WHERE = """
        FROM CFMUDMCC.ROCK_CO_PTS_EXEC_DSHBRD_VW
        WHERE CO_TYPE = 'CO Milestone'
          AND USE_CASE_NO IS NOT NULL
          AND PROJECT_OVERALL_RAG_STATUS <> 'NA'
          AND RAG_STATUS <> 'NA'
          {{GLOBAL_FILTER}}
        """;

    // ---------------- TOP CARD 1 ----------------
    public List<Map<String, Object>> getUseCaseCountByOverallRag(GlobalFilterRequest filter) {
        String sql = """
            SELECT PROJECT_OVERALL_RAG_STATUS,
                   COUNT(DISTINCT USE_CASE_NO) AS useCaseCount
            """ + BASE_WHERE + """
            GROUP BY PROJECT_OVERALL_RAG_STATUS
            """;
        return execute(sql, filter);
    }

    // ---------------- TOP CARD 2 ----------------
    public List<Map<String, Object>> getUseCaseCountByOwnershipAndCategory(GlobalFilterRequest filter) {
        String sql = """
            SELECT USE_CASE_OWNERSHIP,
                   USE_CASE_CATEGORY,
                   COUNT(DISTINCT USE_CASE_NO) AS useCaseCount
            """ + BASE_WHERE + """
            GROUP BY USE_CASE_OWNERSHIP, USE_CASE_CATEGORY
            ORDER BY 3 DESC, 1, 2
            """;
        return execute(sql, filter);
    }

    // ---------------- TOP CARD 3 ----------------
    public List<Map<String, Object>> getMilestoneCountByYearCategoryRag(GlobalFilterRequest filter) {
        String sql = """
            SELECT SUBSTR(DUE_DATE_YEAR, -2) || '-' || USE_CASE_CATEGORY AS dueYearCategory,
                   RAG_STATUS,
                   COUNT(DISTINCT MILESTONE_ID) AS useCaseCount
            """ + BASE_WHERE + """
            GROUP BY SUBSTR(DUE_DATE_YEAR, -2) || '-' || USE_CASE_CATEGORY, RAG_STATUS
            ORDER BY 1
            """;
        return execute(sql, filter);
    }

    // ---------------- COMMON EXECUTOR ----------------
    private List<Map<String, Object>> execute(String sql, GlobalFilterRequest filter) {
        String globalFilter = GlobalFilterBuilder.build(filter);
        sql = sql.replace("{{GLOBAL_FILTER}}", globalFilter);

        Map<String, Object> params = buildParams(filter);
        return jdbcTemplate.queryForList(sql, params);
    }

    private Map<String, Object> buildParams(GlobalFilterRequest f) {
        Map<String, Object> params = new HashMap<>();

        // Use Case
        params.put("useCaseNo", f.getUseCaseNo());
        params.put("useCaseOwnership", f.getUseCaseOwnership());
        params.put("useCaseOwner", f.getUseCaseOwner());
        params.put("useCaseAccountableExecutive", f.getUseCaseAccountableExecutive());
        params.put("useCaseCategory", f.getUseCaseRbcmType());
        params.put("projectOverallRagStatus", f.getUseCaseRagStatus());

        // Owners
        params.put("smtAccountableExecutive", f.getAccountableExecutive());
        params.put("milestoneOwner", f.getMilestoneOwner());
        params.put("deliverableOwner", f.getDeliverableOwner());

        // Timeline
        params.put("milestoneYear", f.getMilestoneYear());
        params.put("deliverableYear", f.getDeliverableYear());

        // Work Effort
        params.put("initiativeName", f.getInitiative());
        params.put("programName", f.getProgram());
        params.put("workEffortName", f.getProject());
        params.put("derivedProgramCategory", f.getProgramCategory());

        // Milestone attributes
        params.put("occConsentOrderArticle", f.getArticle());
        params.put("subPortfolioName", f.getSubPortfolio());

        // RAG
        params.put("milestoneRagStatus", f.getMilestoneRag());
        params.put("deliverableRagStatus", f.getDeliverableRag());

        return params;
    }
}





//global filter 





package com.yourcompany.pts.util;

import com.yourcompany.pts.dto.GlobalFilterRequest;
import java.util.List;

public class GlobalFilterBuilder {

    private GlobalFilterBuilder() {}

    public static String build(GlobalFilterRequest f) {

        StringBuilder sb = new StringBuilder();

        // Use Case
        appendFilter(sb, "USE_CASE_NO", f.getUseCaseNo());
        appendFilter(sb, "USE_CASE_OWNERSHIP", f.getUseCaseOwnership());
        appendFilter(sb, "USE_CASE_OWNER", f.getUseCaseOwner());
        appendFilter(sb, "USE_CASE_ACCOUNTABLE_EXECUTIVE", f.getUseCaseAccountableExecutive());
        appendFilter(sb, "USE_CASE_CATEGORY", f.getUseCaseRbcmType());
        appendFilter(sb, "PROJECT_OVERALL_RAG_STATUS", f.getUseCaseRagStatus());

        // Owners
        appendFilter(sb, "SMT_ACCOUNTABLE_EXECUTIVE", f.getAccountableExecutive());
        appendFilter(sb, "MILESTONE_OWNER", f.getMilestoneOwner());
        appendFilter(sb, "DELIVERABLE_OWNER", f.getDeliverableOwner());

        // Timeline
        appendFilter(sb, "MILESTONE_YEAR", f.getMilestoneYear());
        appendFilter(sb, "DELIVERABLE_YEAR", f.getDeliverableYear());

        // Work Effort
        appendFilter(sb, "INITIATIVE_NAME", f.getInitiative());
        appendFilter(sb, "PROGRAM_NAME", f.getProgram());
        appendFilter(sb, "WORK_EFFORT_NAME", f.getProject());
        appendFilter(sb, "DERIVED_PROGRAM_CATEGORY", f.getProgramCategory());

        // Milestone attributes
        appendFilter(sb, "OCC_CONSENT_ORDER_ARTICLE", f.getArticle());
        appendFilter(sb, "SUBPORTFOLIO_NAME", f.getSubPortfolio());

        // RAG
        appendFilter(sb, "MILESTONE_RAG_STATUS", f.getMilestoneRag());
        appendFilter(sb, "DELIVERABLE_RAG_STATUS", f.getDeliverableRag());

        return sb.toString();
    }

    private static void appendFilter(StringBuilder sb, String column, List<?> values) {
        if (values != null && !values.isEmpty()) {
            sb.append(" AND ").append(column).append(" IN (:").append(column.toLowerCase()).append(")");
        }
    }
}







