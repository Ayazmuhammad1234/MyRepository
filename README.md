
@RestController
@RequestMapping("/api/pts")
@RequiredArgsConstructor
public class PtsDashboardController {

    private final PtsDashboardService ptsDashboardService;

    /**
     * ===============================
     * MAIN PTS DASHBOARD (TABLE DATA)
     * ===============================
     */
    @GetMapping("/dashboard")
    public List<PtsDashboardDto> getDashboardData(
            @RequestParam(required = false) String useCaseNo,
            @RequestParam(required = false) String useCaseOwnership,
            @RequestParam(required = false) String useCaseOwner,
            @RequestParam(required = false) String useCaseAccountableExecutive,
            @RequestParam(required = false) String useCaseRbcmType,
            @RequestParam(required = false) String useCaseRagStatus,

            @RequestParam(required = false) String owner,
            @RequestParam(required = false) String accountableExecutive,
            @RequestParam(required = false) Integer rev,
            @RequestParam(required = false) String milestoneOwner,
            @RequestParam(required = false) String deliverableOwner,

            @RequestParam(required = false) String initiative,
            @RequestParam(required = false) String program,
            @RequestParam(required = false) String project,
            @RequestParam(required = false) String programCategory,
            @RequestParam(required = false) String article,
            @RequestParam(required = false) String subPortfolio,

            @RequestParam(required = false) String milestoneRag,
            @RequestParam(required = false) String deliverableRag,

            // -------- DATE OPTIONS (MANDATORY) --------
            @RequestParam String dateType, // BUSINESS / BASELINE
            @RequestParam LocalDate fromDate,
            @RequestParam LocalDate toDate
    ) {
        return ptsDashboardService.getDashboardData(
                useCaseNo, useCaseOwnership, useCaseOwner,
                useCaseAccountableExecutive, useCaseRbcmType, useCaseRagStatus,
                owner, accountableExecutive, rev,
                milestoneOwner, deliverableOwner,
                initiative, program, project, programCategory,
                article, subPortfolio,
                milestoneRag, deliverableRag,
                dateType, fromDate, toDate
        );
    }

    /**
     * ===============================
     * TOP CARD 1 – PROJECT OVERALL RAG
     * ===============================
     */
    @GetMapping("/top-card/project-rag")
    public List<TopCardProjectRagDto> getTopCardProjectRag(
            @RequestParam String dateType,
            @RequestParam LocalDate fromDate,
            @RequestParam LocalDate toDate,

            @RequestParam(required = false) String useCaseOwnership,
            @RequestParam(required = false) String milestoneRag,
            @RequestParam(required = false) String deliverableRag
    ) {
        return ptsDashboardService.getTopCardProjectRag(
                dateType, fromDate, toDate,
                useCaseOwnership, milestoneRag, deliverableRag
        );
    }

    /**
     * =========================================
     * TOP CARD 2 – OWNERSHIP vs USE CASE CATEGORY
     * =========================================
     */
    @GetMapping("/top-card/ownership-category")
    public List<TopCardOwnershipCategoryDto> getTopCardOwnershipCategory(
            @RequestParam String dateType,
            @RequestParam LocalDate fromDate,
            @RequestParam LocalDate toDate,

            @RequestParam(required = false) String useCaseOwnership,
            @RequestParam(required = false) String useCaseCategory
    ) {
        return ptsDashboardService.getTopCardOwnershipCategory(
                dateType, fromDate, toDate,
                useCaseOwnership, useCaseCategory
        );
    }

    /**
     * ===============================
     * FILTER DROPDOWNS (LANDING)
     * ===============================
     */
    @GetMapping("/filters")
    public PtsFilterResponse getFilterDropdowns() {
        return ptsDashboardService.getAllFilters();
    }
}




public final class GlobalFilterBuilder {

    private GlobalFilterBuilder() {}

    public static String build(Map<String, Object> params) {

        StringBuilder sql = new StringBuilder();

        /* =======================
           1. USE CASE FILTERS
        ======================== */

        if (params.containsKey("useCaseNo")) {
            sql.append(" AND USE_CASE_NO = :useCaseNo");
        }

        if (params.containsKey("useCaseOwnership")) {
            sql.append(" AND USE_CASE_OWNERSHIP = :useCaseOwnership");
        }

        if (params.containsKey("useCaseOwner")) {
            sql.append(" AND USE_CASE_OWNER = :useCaseOwner");
        }

        if (params.containsKey("useCaseAccountableExec")) {
            sql.append(" AND USE_CASE_ACCOUNTABLE_EXECUTIVE = :useCaseAccountableExec");
        }

        if (params.containsKey("useCaseRbcmType")) {
            sql.append(" AND CO_TYPE = :useCaseRbcmType");
        }

        if (params.containsKey("useCaseRagStatus")) {
            sql.append(" AND PROJECT_OVERALL_RAG_STATUS IN (:useCaseRagStatus)");
            params.put("useCaseRagStatus", split(params.get("useCaseRagStatus")));
        }

        /* =======================
           2. OWNER FILTERS
        ======================== */

        if (params.containsKey("accountableExecutive")) {
            sql.append(" AND SMT_ACCOUNTABLE_EXECUTIVE = :accountableExecutive");
        }

        if (params.containsKey("milestoneOwner")) {
            sql.append(" AND CO_TYPE = 'CO Milestone'");
            sql.append(" AND DELIVERABLE_OWNER = :milestoneOwner");
        }

        if (params.containsKey("deliverableOwner")) {
            sql.append(" AND CO_TYPE = 'CO Deliverable'");
            sql.append(" AND DELIVERABLE_OWNER = :deliverableOwner");
        }

        /* =======================
           3. TIMELINE FILTERS
        ======================== */

        if (params.containsKey("timelineType")) {
            sql.append(" AND CO_TYPE = :timelineType");
        }

        /* =======================
           4. WORK EFFORT FILTERS
        ======================== */

        if (params.containsKey("initiative")) {
            sql.append(" AND INITIATIVE_NAME = :initiative");
        }

        if (params.containsKey("program")) {
            sql.append(" AND PROGRAM_NAME = :program");
        }

        if (params.containsKey("project")) {
            sql.append(" AND WORK_EFFORT_NAME = :project");
        }

        if (params.containsKey("programCategory")) {
            sql.append(" AND DERIVED_PROGRAM_CATEGORY = :programCategory");
        }

        /* =======================
           5. MILESTONE ATTRIBUTES
        ======================== */

        if (params.containsKey("article")) {
            sql.append(" AND OCC_CONSENT_ORDER_ARTICLE = :article");
        }

        if (params.containsKey("subPortfolio")) {
            sql.append(" AND SUBPORTFOLIO_NAME = :subPortfolio");
        }

        /* =======================
           6. RAG STATUS FILTERS
        ======================== */

        if (params.containsKey("milestoneRag")) {
            sql.append(" AND CO_TYPE = 'CO Milestone'");
            sql.append(" AND RAG_STATUS IN (:milestoneRag)");
            params.put("milestoneRag", split(params.get("milestoneRag")));
        }

        if (params.containsKey("deliverableRag")) {
            sql.append(" AND CO_TYPE = 'CO Deliverable'");
            sql.append(" AND RAG_STATUS IN (:deliverableRag)");
            params.put("deliverableRag", split(params.get("deliverableRag")));
        }

        return sql.toString();
    }

    private static List<String> split(Object value) {
        return Arrays.stream(value.toString().split(","))
                .map(String::trim)
                .toList();
    }
}

_____________________________________________


@Data
public class TopCardFilterRequest {

    // Use Case
    private String useCaseOwnership;
    private String useCaseOwner;
    private String useCaseAccountableExecutive;
    private String useCaseRagStatus;

    // Owners
    private String accountableExecutive;
    private String milestoneOwner;
    private String deliverableOwner;

    // Work Effort
    private String initiative;
    private String program;
    private String project;
    private String programCategory;

    // Milestone Attributes
    private String article;
    private String subPortfolio;

    // RAG
    private String milestoneRag;
    private String deliverableRag;

    // Timeline
    private LocalDate milestoneFromDate;
    private LocalDate milestoneToDate;
}


@Data
@AllArgsConstructor
public class TopCard1Response {

    private String ragStatus;
    private Long useCaseCount;
}



@Repository
@RequiredArgsConstructor
public class TopCard1Repository {

    private final NamedParameterJdbcTemplate jdbcTemplate;

    private static final String SQL = """
        WITH preprocessed_data AS (
            SELECT
                USE_CASE_NO,
                PROJECT_OVERALL_RAG_STATUS,
                USE_CASE_OWNERSHIP,
                USE_CASE_OWNER,
                USE_CASE_ACCOUNTABLE_EXECUTIVE,
                PROGRAM_NAME,
                INITIATIVE_NAME,
                WORK_EFFORT_NAME,
                DERIVED_PROGRAM_CATEGORY,
                OCC_CONSENT_ORDER_ARTICLE,
                SUBPORTFOLIO_NAME,
                RAG_STATUS,
                DUE_DATE,
                CO_TYPE
            FROM CFMUDMCC.ROCK_CO_PTS_EXEC_DSHBRD_VW
            WHERE USE_CASE_NO IS NOT NULL
              AND PROJECT_OVERALL_RAG_STATUS <> 'NA'
              AND RAG_STATUS <> 'NA'
              {{GLOBAL_FILTER}}
        )
        SELECT
            PROJECT_OVERALL_RAG_STATUS AS RAG_STATUS,
            COUNT(DISTINCT USE_CASE_NO) AS USE_CASE_COUNT
        FROM preprocessed_data
        GROUP BY PROJECT_OVERALL_RAG_STATUS
        ORDER BY PROJECT_OVERALL_RAG_STATUS
        """;

    public List<TopCard1Response> fetchTopCard1(Map<String, Object> params) {

        String finalSql = GlobalFilterBuilder.build(SQL, params);

        return jdbcTemplate.query(
                finalSql,
                params,
                (rs, rowNum) -> new TopCard1Response(
                        rs.getString("RAG_STATUS"),
                        rs.getLong("USE_CASE_COUNT")
                )
        );
    }
}






@Service
@RequiredArgsConstructor
public class TopCard1Service {

    private final TopCard1Repository repository;

    public List<TopCard1Response> getTopCard1(TopCardFilterRequest request) {

        Map<String, Object> params = FilterParamMapper.map(request);

        return repository.fetchTopCard1(params);
    }
}

@RestController
@RequestMapping("/api/pts/dashboard")
@RequiredArgsConstructor
public class TopCard1Controller {

    private final TopCard1Service service;

    @PostMapping("/topcard1")
    public ResponseEntity<List<TopCard1Response>> getTopCard1(
            @RequestBody TopCardFilterRequest request) {

        return ResponseEntity.ok(service.getTopCard1(request));
    }
}



