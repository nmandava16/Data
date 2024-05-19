import org.springframework.stereotype.Service;
import @RestController
@RequestMapping("/api/dealers")
public class CostAndGrossController {

    @Autowired
    private DealerService dealerService;

    @PostMapping(value = "/{dealerId}/marketing/leads", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Void> saveCostAndGrossForLeads(
            @RequestBody DealerCostAndGross dealerRequest,
            @PathVariable("dealerId") int dealerId) {
        return dealerService.processDealerData(dealerId, dealerRequest);
    }
}org.springframework.transaction.annotation.Transactional;
import org.springframework.http.ResponseEntity;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;
import java.util.stream.Collectors;
import java.util.ArrayList;
import java.util.Set;
import java.util.HashSet;

@Service
public class DealerService {

    private static final Logger log = LoggerFactory.getLogger(DealerService.class);

    private final CostAndGrossDetailsRepository costAndGrossDetailsRepository;

    public DealerService(CostAndGrossDetailsRepository costAndGrossDetailsRepository) {
        this.costAndGrossDetailsRepository = costAndGrossDetailsRepository;
    }

    @Transactional
    public ResponseEntity<Void> processDealerData(int dealerId, DealerCostAndGross dealerCostAndGross) {
        try {
            // Add Feasible Validations
            validateInput(dealerId, dealerCostAndGross);

            List<DealerLeadSourceCostAndGrossEntity> entities = dealerCostAndGross.getLeadSources().stream()
                    .map(lead -> mapToEntity(lead, dealerId))
                    .collect(Collectors.toList());

            // Find the events for monitoring
            log.info("Processed {} lead sources for dealer ID {}", entities.size(), dealerId);

            costAndGrossDetailsRepository.saveAll(entities);

            return ResponseEntity.ok().build();
        } catch (Exception e) {
            // Handle exception, log error, etc.
            log.error("Error while processing dealer data", e);
            return ResponseEntity.internalServerError().build();
        }
    }

    private void validateInput(int dealerId, DealerCostAndGross dealerCostAndGross) {
        if (dealerCostAndGross == null || dealerCostAndGross.getLeadSources() == null || dealerCostAndGross.getLeadSources().isEmpty()) {
            throw new IllegalArgumentException("DealerCostAndGross and its LeadSources cannot be null or empty");
        }

        if (dealerId <= 0) {
            throw new IllegalArgumentException("Dealer ID must be a positive integer");
        }

        Set<Integer> leadSourceIds = new HashSet<>();
        for (LeadSource lead : dealerCostAndGross.getLeadSources()) {
            if (lead.getLeadSourceId() <= 0) {
                throw new IllegalArgumentException("LeadSource ID must be a positive integer");
            }
            if (lead.getCost() < 0) {
                throw new IllegalArgumentException("Cost cannot be negative");
            }
            if (lead.getGross() < 0) {
                throw new IllegalArgumentException("Gross cannot be negative");
            }
            if (!leadSourceIds.add(lead.getLeadSourceId())) {
                throw new IllegalArgumentException("Duplicate LeadSource ID found: " + lead.getLeadSourceId());
            }
        }
