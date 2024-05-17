@RestController
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
}


import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> handleIllegalArgumentException(IllegalArgumentException ex, WebRequest request) {
        return new ResponseEntity<>(ex.getMessage(), HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleException(Exception ex, WebRequest request) {
        return new ResponseEntity<>("An internal error occurred. Please try again later.", HttpStatus.INTERNAL_SERVER_ERROR);
    }
}



import org.springframework.http.ResponseEntity;

public class DealerService {

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
        } catch (IllegalArgumentException e) {
            log.error("Error while processing dealer data", e);
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(e.getMessage());
        } catch (Exception e) {
            log.error("Error while processing dealer data", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
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
        for (Lead lead : dealerCostAndGross.getLeadSources()) {
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
    }

    private DealerLeadSourceCostAndGrossEntity mapToEntity(Lead lead, int dealerId) {
        DealerLeadSourceCostAndGrossEntity entity = new DealerLeadSourceCostAndGrossEntity();
        entity.setCost(lead.getCost());
        entity.setGross(lead.getGross());
        entity.setLeadSourceId(lead.getLeadSourceId());
        entity.setInternalDealerId(dealerId);
        return entity;
    }
}
