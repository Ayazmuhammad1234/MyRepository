
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
