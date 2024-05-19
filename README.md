
import com.capitolone.le.costandgross.TestData;
import com.capitolone.le.costandgross.entity.DealerLeadSourceCostAndGrossEntity;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class CostAndGrossDetailsRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private CostAndGrossDetailsRepository repository;

    @Test
    public void testFindingCostAndGrossByDealerIdAndMonth() {
        // Assuming that TestData.DEALER_COST_AND_GROSS is an entity or can be adapted to an entity
        DealerLeadSourceCostAndGrossEntity entity = new DealerLeadSourceCostAndGrossEntity();
        entity.setDealerId(TestData.DEALER_ID);
        entity.setMonth("May");
        entity.setYear(2023);
        entityManager.persist(entity);
        entityManager.flush();

        // Retrieving from the database to verify it's correctly stored
        DealerLeadSourceCostAndGrossEntity foundEntity = repository.findByDealerIdAndMonth(TestData.DEALER_ID, "May").get(0);
        assertThat(foundEntity).isNotNull();
        assertThat(foundEntity.getDealerId()).isEqualTo(entity.getDealerId());
        assertThat(foundEntity.getMonth()).isEqualTo(entity.getMonth());
    }
}
