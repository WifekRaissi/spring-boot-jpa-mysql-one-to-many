# spring-boot-jpa-mysql-one-to-many
 
 On a commencé dans le tutorial précédent l'intégration d'une base de données(MySQL) en utilisant Spring Data et Hibernate.
 https://github.com/WifekRaissi/spring-boot-jpa-mysql
 
On continuera avec MySQL en étudiant le mapping entre les tables et on commence par la relation One to Many entre la table Salarie et une nouvelle table Departement.

L'architecture de l'application est la suivante.


![alt text](https://github.com/WifekRaissi/spring-boot-jpa-mysql-one-to-many/blob/master/src/main/resources/images/architecture.PNG)       

 
  ## Création de la table Departement:
  
 ```
  CREATE TABLE department (
        id bigint(20) NOT NULL AUTO_INCREMENT,
        description varchar(256) NOT NULL,
        nom varchar(256) NOT NULL,
        PRIMARY KEY (`id`));
 ```
 

 

## Departement.java

```

import javax.persistence.*;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "department")

public class Departement {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotNull
    @Size(max = 256)
    private String nom;
    @NotNull
    @Size(max = 256)
    private String description;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getNom() {
        return nom;
    }

    public void setNom(String nom) {
        this.nom = nom;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
    @OneToMany(cascade = CascadeType.ALL,
            fetch = FetchType.LAZY,
            mappedBy = "departement")
    private Set<Salarie> salaries = new HashSet<>();

    public Set<Salarie> getSalaries() {
        return salaries;
    }

    public void setSalaries(Set<Salarie> salaries) {
        this.salaries = salaries;
    }
```

@OneToMany : indique qu'il s'agit d'une relation bidirectionnelle avec Salarie mappée par Departement.

cascade = CascadeType.ALL: indique que Salarie dépend du departement pour toutes les opérations(ajout, modification, supression,..) par exemple si on supprime un département on va supprimer automatiquement tous ses salariés.

 fetch = FetchType.LAZY: indique que la relation doit être chargée à la demande. 
 
## Salarie.java

```
import com.fasterxml.jackson.annotation.JsonIgnore;
import org.springframework.beans.factory.annotation.Required;

import javax.persistence.*;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import java.math.BigDecimal;
import java.util.concurrent.atomic.AtomicInteger;

@Entity
@Table(name = "salarie")
public class Salarie {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private static final AtomicInteger count = new AtomicInteger(0);

    @NotEmpty
    @NotNull
    private String nom;

    @NotEmpty
    @NotNull
    private String prenom;

    @NotNull
    private BigDecimal salaire;

    @NotEmpty
    @NotNull
    @Size(max = 256, message = "address should have maximum 256 characters")
    private String adresse;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id", nullable = false)
    private Departement departement;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getNom() {
        return nom;
    }

    public void setNom(String nom) {
        this.nom = nom;
    }

    public String getPrenom() {
        return prenom;
    }

    public void setPrenom(String prenom) {
        this.prenom = prenom;
    }

    public BigDecimal getSalaire() {
        return salaire;
    }

    public void setSalaire(BigDecimal salaire) {
        this.salaire = salaire;
    }

    public String getAdresse() {
        return adresse;
    }
    @JsonIgnore
    public Departement getDepartement() {
        return departement;
    }

    public void setDepartement(Departement departement) {
        this.departement = departement;
    }

    @Required
    public void setAdresse(String adresse) {
        this.adresse = adresse;
    }

}
    
```
    
  ##   Repository
  ##      DepartementRepository.java
  ```
  
import com.axeane.SpringBootMysql.model.Departement;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface DepartementRepository extends JpaRepository<Departement, Long> {

    List<Departement> findDepartementByNom(String nom);

    Departement findDepartementById(Long id);
}

  ```
  
  
   ##        SalarieRepository.java
   
   ```
import com.axeane.SpringBootMysql.model.Salarie;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
@Repository
public interface SalariesRepository extends JpaRepository<Salarie, Long> {
    List<Salarie> findSalarieByNom(String nom);

    Salarie findSalarieById(Long id);
    Page<Salarie> findByDepartementId(Long departementId, Pageable pageable);

}
   ```
   
   
   
   ##    Service
   
   ##       DepartementServiceImpl.java
   
   ```
  import com.axeane.SpringBootMysql.model.Departement;
import com.axeane.SpringBootMysql.repositories.DepartementRepository;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

@Service
public class DepartementServiceImpl implements DepartementService {
    private final DepartementRepository departementRepository;

    public DepartementServiceImpl(DepartementRepository departementRepository) {
        this.departementRepository = departementRepository;
    }

    @Override
    public Page<Departement> getAllDepartments(Pageable pageable) {
        return departementRepository.findAll(pageable);
    }

    @Override
    public void createDepartment(Departement departement) {
        departementRepository.save(departement);
    }

    @Override
    public void updateDepartement(Departement departementRequest) {
        departementRepository.findById(departementRequest.getId()).map(departement -> {
            departement.setNom(departementRequest.getNom());
            departement.setDescription(departementRequest.getDescription());
            return departementRepository.save(departement);
        });
    }

    @Override
    public void deleteDepartment(Long departementId) {
        departementRepository.findById(departementId).map(departement -> {
            departementRepository.delete(departement);
            return departement;
        });
    }

}

   ```
   
  ##   SalarieServiceImpl.java
 
 ```
 
import com.axeane.SpringBootMysql.ResourceNotFoundException;
import com.axeane.SpringBootMysql.model.Salarie;
import com.axeane.SpringBootMysql.repositories.DepartementRepository;
import com.axeane.SpringBootMysql.repositories.SalariesRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.*;


@Service
public class SalariesServiceImpl implements SalariesService {
    private final SalariesRepository salariesRepository;
    private final DepartementRepository departementRepository;

    public SalariesServiceImpl(SalariesRepository salariesRepository, DepartementRepository departementRepository) {
        this.salariesRepository = salariesRepository;
        this.departementRepository = departementRepository;
    }

    private Logger logger = LoggerFactory.getLogger(SalariesServiceImpl.class);

    @Override
    public void addsalarie(Long departementId, Salarie salarie) {
        departementRepository.findById(departementId).map(departement -> {
            salarie.setDepartement(departement);
            return salariesRepository.save(salarie);
        });
    }

    @Override
    public Page<Salarie> getAllSalariesByDepartementtId(Long departementId,
                                                        Pageable pageable) {
        return salariesRepository.findByDepartementId(departementId, pageable);
    }

    @Override
    public void deleteSalaried(Long departementId, Long salarieId) {
        if (!departementRepository.existsById(departementId)) {
            throw new ResourceNotFoundException("departementId" + departementId + " not found");
        }
        salariesRepository.findById(salarieId).map(salarie -> {
            salariesRepository.delete(salarie);
            return salarie;
        });
    }
    @Override
    public void updateSalarie(Long departementId, Salarie salarieRequest) {
        if (!departementRepository.existsById(departementId)) {
            throw new ResourceNotFoundException("departementId " + departementId + " not found");
        }
        salariesRepository.findById(salarieRequest.getId()).map(salarie -> {
            salarie.setNom(salarieRequest.getNom());
            salarie.setPrenom(salarieRequest.getPrenom());
            salarie.setAdresse(salarieRequest.getAdresse());
            salarie.setSalaire(salarieRequest.getSalaire());
            return salariesRepository.save(salarie);
        });
    }

    @Override
    public Salarie findSalarieById(Long id) {
        return salariesRepository.findSalarieById(id);
    }

 ```
  ## Controllers
      
  ##   DepartementController.java
  
  ```
  
import com.axeane.SpringBootMysql.model.Departement;
import com.axeane.SpringBootMysql.services.DepartementService;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiResponse;
import io.swagger.annotations.ApiResponses;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;

@RestController
public class DepartementController {


    private final DepartementService departementService;

    public DepartementController(DepartementService departementService) {
        this.departementService = departementService;
    }

    @ApiOperation(value = "the list of departments", response = Iterable.class)
    @ApiResponses(value = {
            @ApiResponse(code = 200, message = "Successfully retrieved list"),
            @ApiResponse(code = 404, message = "The resource you were trying to reach is not found")
    }
    )
    @GetMapping("/departements")
    public ResponseEntity getAllDepartments(Pageable pageable) {
        Page<Departement> departements = departementService.getAllDepartments(pageable);
        return new ResponseEntity<>(departements, HttpStatus.OK);
    }

    @ApiOperation(value = "add a new department")
    @ApiResponses(value = {
            @ApiResponse(code = 201, message = "Successfully created department")}
    )
    @ResponseStatus(HttpStatus.CREATED)
    @PostMapping("/departements")
    public ResponseEntity createDepartment(@Valid @RequestBody Departement departement) {
        departementService.createDepartment(departement);
        return new ResponseEntity<>(departement, HttpStatus.CREATED);
    }

    @ApiOperation(value = "update a department")
    @ApiResponses(value = {
            @ApiResponse(code = 200, message = "Successfully updated department"),
            @ApiResponse(code = 404, message = "The resource you were trying to reach is not found")}
    )
    @PutMapping("/departements")
    public ResponseEntity updateDepartement(@Valid @RequestBody Departement departement) {
        departementService.updateDepartement(departement);
        return new ResponseEntity<>(departement, HttpStatus.OK); }

    @ApiOperation(value = "delete a department")
    @ApiResponses(value = {
            @ApiResponse(code = 200, message = "Successfully deleted department"),
            @ApiResponse(code = 404, message = "The resource you were trying to reach is not found")})
    @DeleteMapping("/departements/{departementId}")
    public ResponseEntity deleteDepartment(@PathVariable(value = "departementId") Long departementId) {
        departementService.deleteDepartment(departementId);
        return new ResponseEntity(HttpStatus.OK);
    }
  
  ```
  
  ## SalariesController.java
  
  
  ```
import com.axeane.SpringBootMysql.model.Salarie;
import com.axeane.SpringBootMysql.services.SalariesService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiResponse;
import io.swagger.annotations.ApiResponses;
import org.springframework.boot.autoconfigure.jackson.Jackson2ObjectMapperBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.zalando.problem.ProblemModule;
import org.zalando.problem.validation.ConstraintViolationProblemModule;

import javax.validation.Valid;

@RestController
@Api(value = "gestion des salariés", description = "Operations pour la gestion des salariés")
public class SalariesController {

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer problemObjectMapperModules() {
        return jacksonObjectMapperBuilder -> jacksonObjectMapperBuilder.modules(
                new ProblemModule(),
                new ConstraintViolationProblemModule());
    }

    private final SalariesService salariesService;

    public SalariesController(SalariesService salariesService) {
        this.salariesService = salariesService;
    }

    @ApiOperation(value = "View a list of salaries by department", response = Iterable.class)
    @ApiResponses(value = {
            @ApiResponse(code = 200, message = "Successfully retrieved list"),
            @ApiResponse(code = 404, message = "The resource you were trying to reach is not found")}
    )
    @GetMapping("/departements/{departementId}/salaries")
    public ResponseEntity getAllSalariesByDepartementId(@PathVariable(value = "departementId") Long departementId, Pageable pageable) {
        Page<Salarie> salaries = salariesService.getAllSalariesByDepartementtId(departementId, pageable);
        return new ResponseEntity<>(salaries, HttpStatus.OK);
    }

    @ApiOperation(value = "add a new salaried")
    @PostMapping("/departements/{departementId}/salaries")
    @ApiResponses(value = {
            @ApiResponse(code = 201, message = "Successfully created salaried")}
    )
    @ResponseStatus(HttpStatus.CREATED)
    public ResponseEntity createSalarie(@PathVariable(value = "departementId") Long departementId,
                                        @Valid @RequestBody Salarie salarie) {
        salariesService.addsalarie(departementId, salarie);
        return new ResponseEntity<>(salarie, HttpStatus.CREATED);
    }

    @ApiOperation(value = "update a salaried")
    @ApiResponses(value = {
            @ApiResponse(code = 200, message = "Successfully updated salaried"),
            @ApiResponse(code = 404, message = "The resource you were trying to reach is not found")}
    )
    @PutMapping("/departements/{departementId}/salaries/{salarieId}")
    public ResponseEntity updateSalarie(@PathVariable(value = "departementId") Long departementId,
                                        @Valid @RequestBody Salarie salarieRequest) {
        salariesService.updateSalarie(departementId, salarieRequest);
        return new ResponseEntity<>(salarieRequest, HttpStatus.OK);
    }

    @ApiOperation(value = "delete a salaried")
    @ApiResponses(value = {
            @ApiResponse(code = 200, message = "Successfully deleted salaried"),
            @ApiResponse(code = 404, message = "The resource you were trying to reach is not found")
    })
    @DeleteMapping("/departements/{departementId}/salaries/{salarieId}")
    public ResponseEntity deleteSalarie(@PathVariable(value = "departementId") Long departementId,
                                        @PathVariable(value = "salarieId") Long salarieId) {
        salariesService.deleteSalaried(departementId, salarieId);
        return new ResponseEntity(HttpStatus.OK);
    }
    @GetMapping("/salaries/{id}")
    public ResponseEntity findSalarie(@PathVariable(value = "id") Long id) {
       Salarie salarie =salariesService.findSalarieById(id);

        return new ResponseEntity<>(salarie,HttpStatus.OK);
    }
}
  ```

 ## Tests avec Postman:
 
 Ajout d'un nouveau salarié à un département.
 
  ![alt text](https://github.com/WifekRaissi/spring-boot-jpa-mysql-one-to-many/blob/master/src/main/resources/images/ajoutSalarie.PNG) 
  
  Liste des salariés par département.
  
  ![alt text](https://github.com/WifekRaissi/spring-boot-jpa-mysql-one-to-many/blob/master/src/main/resources/images/listSalariesparDepartement.PNG) 

  
  ## Conclusion
  On réalisé dans ce tutorial le mapping entre les tables Salarie et departement par la relation One to Many. On passe au mapping de la relation One to One dans le tutorial suivant.

https://github.com/WifekRaissi/spring-boot-jpa-mysql-one-to-one


