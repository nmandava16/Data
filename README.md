import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.sql.Timestamp;
import java.util.List;
import java.util.Arrays;

public class DealerServiceTest {

    @Mock
    private CostAndGrossDetailsRepository repository; // Ensure this is the correct repository interface

    @InjectMocks
    private DealerService service;

    @BeforeEach
    public void init() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testGetDealerWithLeads() {
        // Arrange
        Long dealerId = TestData.DEALER_ID;
        String month = "May";
        Integer year = 2023;

        DealerLeadSourceCostAndGrossEntity entity1 = createEntity(dealerId, "Online", month, 100.00, 150.00, 50.00, 10);
        DealerLeadSourceCostAndGrossEntity entity2 = createEntity(dealerId, "Referral", month, 200.00, 250.00, 100.00, 20);

        List<DealerLeadSourceCostAndGrossEntity> expectedEntities = Arrays.asList(entity1, entity2);

        when(repository.findCostAndGrossByDealerIdAndMonth(dealerId, month, year))
            .thenReturn(expectedEntities);

        // Act
        List<DealerLeadSourceCostAndGrossEntity> actualEntities = service.getDealerWithLeads(dealerId, month, year);

        // Assert
        assertNotNull(actualEntities, "The returned list should not be null.");
        assertEquals(expectedEntities.size(), actualEntities.size(), "The number of entities should match.");
        assertTrue(actualEntities.containsAll(expectedEntities), "The actual list should contain all the expected entities.");
        verify(repository, times(1)).findCostAndGrossByDealerIdAndMonth(dealerId, month, year);
    }

    private DealerLeadSourceCostAndGrossEntity createEntity(Long dealerId, String leadSource, String month, double cost, double gross, double net, long sales) {
        DealerLeadSourceCostAndGrossEntity entity = new DealerLeadSourceCostAndGrossEntity();
        entity.setInternalDealerId(dealerId);
        entity.setModifiedLeadSource(leadSource);
        entity.setMonth(month);
        entity.setCost(cost);
        entity.setGross(gross);
        entity.setNet(net);
        entity.setNumberOfSales(sales);
        entity.setCreatedDate(new Timestamp(System.currentTimeMillis()));
        entity.setUpdatedDate(new Timestamp(System.currentTimeMillis()));
        entity.setUserUpdated("testUser");
        entity.setVersionNumber(1);
        return entity;
    }
}
