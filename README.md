import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.util.List;
import java.util.Arrays;

public class DealerServiceTest {

    @Mock
    private CostAndGrossDetailsRepository repository; 

    @InjectMocks
    private DealerService service;

    @BeforeEach
    public void init() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testGetDealerWithLeads() {
        // Arrange
        int internalDealerId = 1;
        String reportMonth = "May";
        int reportYear = 2023;
        int offset = 0;
        int limit = 10;

        DealerLeadSourceCostAndGrossEntity entity1 = new DealerLeadSourceCostAndGrossEntity();
        entity1.setInternalDealerId(internalDealerId);
        entity1.setModifiedLeadSource("Online");
        entity1.setMonth(reportMonth);
        entity1.setCost(100.00);
        entity1.setGross(150.00);
        entity1.setNet(50.00);
        entity1.setNumberOfSales(10);

        DealerLeadSourceCostAndGrossEntity entity2 = new DealerLeadSourceCostAndGrossEntity();
        entity2.setInternalDealerId(internalDealerId);
        entity2.setModifiedLeadSource("Referral");
        entity2.setMonth(reportMonth);
        entity2.setCost(200.00);
        entity2.setGross(250.00);
        entity2.setNet(100.00);
        entity2.setNumberOfSales(20);

        List<DealerLeadSourceCostAndGrossEntity> entities = Arrays.asList(entity1, entity2);

        when(repository.findCostAndGrossByInternalDealerIdAndReportMonth(internalDealerId, reportMonth))
            .thenReturn(entities);

        // Act
        DealerCostAndGrossResponse response = service.getDealerWithLeads(internalDealerId, reportMonth, reportYear, offset, limit);

        // Assert
        assertNotNull(response, "The response should not be null.");
        assertNotNull(response.getLeadSources(), "Lead sources should not be null.");
        assertEquals(2, response.getLeadSources().size(), "Should return 2 leads.");
        assertTrue(response.getPagination().getCount() == 2, "Pagination count should match the number of records.");
    }
}
