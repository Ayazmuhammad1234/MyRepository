
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
