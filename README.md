WITH preprocessed_data AS (
    SELECT
        USE_CASE_NO,
        USE_CASE_OWNERSHIP,
        USE_CASE_OWNER,
        USE_CASE_ACCOUNTABLE_EXECUTIVE,
        USE_CASE_CATEGORY,
        PROJECT_OVERALL_RAG_STATUS,
        SMT_ACCOUNTABLE_EXECUTIVE,
        DELIVERABLE_OWNER,
        PROGRAM_NAME,
        OCC_CONSENT_ORDER_ARTICLE,
        SUBPORTFOLIO_NAME,
        RAG_STATUS,
        DUE_DATE,
        DERIVED_PROGRAM_CATEGORY,
        WORK_EFFORT_NAME,
        INITIATIVE_NAME,
        MILESTONE_ID,
        CO_TYPE
    FROM CFMUDMCC.ROCK_CO_PTS_EXEC_DSHBRD_VW
    WHERE USE_CASE_NO IS NOT NULL
      AND PROJECT_OVERALL_RAG_STATUS <> 'NA'
      AND RAG_STATUS <> 'NA'
)

SELECT DISTINCT
    MS.USE_CASE_NO,
    MS.USE_CASE_OWNERSHIP,
    MS.USE_CASE_OWNER,
    MS.USE_CASE_ACCOUNTABLE_EXECUTIVE,
    MS.USE_CASE_CATEGORY,
    MS.PROJECT_OVERALL_RAG_STATUS,

    UCDate.USE_CASE_DUE_DT_YEAR,
    UCDate.USE_CASE_DUE_DT_QTR,
    UCDate.USE_CASE_DUE_DT_MONTH,

    MS.SMT_ACCOUNTABLE_EXECUTIVE,
    MS.MILESTONE_OWNER,
    Del.DELIVERABLE_OWNER,

    MS.MILESTONE_YEAR,
    MS.MILESTONE_QTR,
    MS.MILESTONE_MONTH,
    Del.DELIVERABLE_YEAR,
    Del.DELIVERABLE_QTR,
    Del.DELIVERABLE_MONTH,

    Del.INITIATIVE_NAME,
    MS.PROGRAM_NAME,
    Del.WORK_EFFORT_NAME,
    Del.DERIVED_PROGRAM_CATEGORY,

    MS.OCC_CONSENT_ORDER_ARTICLE,
    MS.SUBPORTFOLIO_NAME,
    MS.MILESTONE_RAG_STATUS,
    Del.DELIVERABLE_RAG_STATUS
FROM
(
    SELECT
        MILESTONE_ID,
        DELIVERABLE_OWNER,
        INITIATIVE_NAME,
        WORK_EFFORT_NAME,
        DERIVED_PROGRAM_CATEGORY,
        RAG_STATUS AS DELIVERABLE_RAG_STATUS,
        EXTRACT(YEAR FROM DUE_DATE) AS DELIVERABLE_YEAR,
        'Q' || TO_CHAR(DUE_DATE, 'Q') || ' ' || TO_CHAR(DUE_DATE, 'YYYY') AS DELIVERABLE_QTR,
        TO_CHAR(DUE_DATE, 'Mon YYYY') AS DELIVERABLE_MONTH
    FROM preprocessed_data
    WHERE CO_TYPE = 'CO Deliverable'
) Del
LEFT JOIN
(
    SELECT
        MILESTONE_ID,
        USE_CASE_NO,
        USE_CASE_OWNERSHIP,
        USE_CASE_OWNER,
        USE_CASE_ACCOUNTABLE_EXECUTIVE,
        USE_CASE_CATEGORY,
        PROJECT_OVERALL_RAG_STATUS,
        SMT_ACCOUNTABLE_EXECUTIVE,
        DELIVERABLE_OWNER AS MILESTONE_OWNER,
        PROGRAM_NAME,
        OCC_CONSENT_ORDER_ARTICLE,
        SUBPORTFOLIO_NAME,
        RAG_STATUS AS MILESTONE_RAG_STATUS,
        EXTRACT(YEAR FROM DUE_DATE) AS MILESTONE_YEAR,
        'Q' || TO_CHAR(DUE_DATE, 'Q') || ' ' || TO_CHAR(DUE_DATE, 'YYYY') AS MILESTONE_QTR,
        TO_CHAR(DUE_DATE, 'Mon YYYY') AS MILESTONE_MONTH
    FROM preprocessed_data
    WHERE CO_TYPE = 'CO Milestone'
) MS
    ON Del.MILESTONE_ID = MS.MILESTONE_ID
LEFT JOIN
(
    SELECT
        USE_CASE_NO,
        EXTRACT(YEAR FROM DUE_DATE) AS USE_CASE_DUE_DT_YEAR,
        'Q' || TO_CHAR(DUE_DATE, 'Q') || ' ' || TO_CHAR(DUE_DATE, 'YYYY') AS USE_CASE_DUE_DT_QTR,
        TO_CHAR(DUE_DATE, 'Mon YYYY') AS USE_CASE_DUE_DT_MONTH
    FROM preprocessed_data
    WHERE CO_TYPE = 'CO Milestone'
      AND USE_CASE_CATEGORY = 'End Point Consumption'
) UCDate
    ON MS.USE_CASE_NO = UCDate.USE_CASE_NO;


@Data
public class PtsDashboardDto {

    private String useCaseNo;
    private String useCaseOwnership;
    private String useCaseOwner;
    private String useCaseAccountableExecutive;
    private String useCaseCategory;
    private String projectOverallRagStatus;

    private Integer useCaseDueDtYear;
    private String useCaseDueDtQtr;
    private String useCaseDueDtMonth;

    private String smtAccountableExecutive;
    private String milestoneOwner;
    private String deliverableOwner;

    private Integer milestoneYear;
    private String milestoneQtr;
    private String milestoneMonth;

    private Integer deliverableYear;
    private String deliverableQtr;
    private String deliverableMonth;

    private String initiativeName;
    private String programName;
    private String workEffortName;
    private String derivedProgramCategory;


    private String occConsentOrderArticle;
    private String subportfolioName;
    private String milestoneRagStatus;
    private String deliverableRagStatus;
}

@Repository
public class PtsDashboardRepository {

    @Autowired
    private NamedParameterJdbcTemplate jdbcTemplate;

    @Value("classpath:sql/pts_dashboard.sql")
    private Resource ptsDashboardSql;

    public List<PtsDashboardDto> fetchPtsDashboard() {
        try (InputStream is = ptsDashboardSql.getInputStream()) {

            String sql = new BufferedReader(
                    new InputStreamReader(is, StandardCharsets.UTF_8))
                    .lines()
                    .collect(Collectors.joining("\n"));

            return jdbcTemplate.query(
                    sql,
                    new BeanPropertyRowMapper<>(PtsDashboardDto.class)
            );

        } catch (Exception e) {
            throw new RuntimeException("Failed to read PTS SQL file", e);
        }
    }
}


@Service
public class PtsDashboardService {

    @Autowired
    private PtsDashboardRepository repository;

    public List<PtsDashboardDto> getDashboardData() {
        return repository.fetchPtsDashboard();
    }
}


@RestController
@RequestMapping("/api/pts-dashboard")
public class PtsDashboardController {

    @Autowired
    private PtsDashboardService service;

    @GetMapping
    public ResponseEntity<List<PtsDashboardDto>> getPtsDashboard() {
        return ResponseEntity.ok(service.getDashboardData());
    }
}



    
