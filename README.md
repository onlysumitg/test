# test
package application;

import javafx.collections.FXCollections;
import javafx.collections.ObservableList;
import javafx.event.Event;
import javafx.event.EventHandler;
import javafx.fxml.FXML;
import javafx.scene.control.*;
import javafx.scene.control.TableColumn.CellEditEvent;
import javafx.scene.control.cell.ComboBoxTableCell;
import javafx.scene.control.cell.PropertyValueFactory;
import javafx.scene.control.cell.TextFieldTableCell;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.input.Clipboard;
import javafx.scene.input.ClipboardContent;
import javafx.scene.input.MouseButton;
import javafx.scene.input.MouseEvent;
import javafx.util.Callback;
import util.BreakPath;

import java.io.File;
import java.net.URI;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.Date;

import directorytree.*;
import filelist.FileEntry;
import filelist.FileEntryDAO;
import filelist.FileList;

public class MainController {

	private ObservableList<FileEntry> data = FXCollections.observableArrayList();
	private ObservableList<FileEntry> searchData = FXCollections.observableArrayList();
	private ObservableList<String> categories = FXCollections.observableArrayList();
	private ObservableList<String> types = FXCollections.observableArrayList();

	// public Image fileImage=new
	// Image(getClass().getResourceAsStream("text-x-generic.png"));
	
	@FXML
	private ImageView gmarkLogo;
	
	@FXML
	private Hyperlink gmarkLink;
	
	@FXML
	private Hyperlink emailLink;
	
	@FXML
	private Accordion accordionList;
	
	@FXML
	private TitledPane optionsAccordion;
	
	@FXML
	private TitledPane fileListAccordion;
	
	
	@FXML
	private ImageView screenLogo;
	
	@FXML
	private TableView<FileEntry> fileList;
	

	@FXML
	private TableColumn<FileEntry, Path> fileName;

	@FXML
	private TableColumn<FileEntry, String> fileType;

	@FXML
	private TableColumn<FileEntry, String> fileTag;

	@FXML
	private TableColumn<FileEntry, String> fileCategory;

	@FXML
	private TableColumn<FileEntry, Date> fileCreated;
	@FXML
	private TableColumn<FileEntry, Date> fileModified;
	
	
	private TableColumn<FileEntry, String> fileLocation = new TableColumn<FileEntry, String>("File Location");

	@FXML
	private ComboBox<String> searchCategory;

	@FXML
	private ComboBox<String> searchType;

	@FXML
	private TreeView<String> treeView;

	@FXML
	private TextField searchBox;

	@FXML
	private Button searchButton;

	@FXML
	private TextField message;

	private void setHeader() {
		
		screenLogo.setImage(new Image("file:resources/lookup.png"));
		searchBox.promptTextProperty().set("Tell me what you are looking for ?");
		searchButton.setGraphic(new ImageView(new Image("file:resources/loupe.png")));

		FileEntryDAO.loadDistinct("category", categories);
		FileEntryDAO.loadDistinct("type", types);

		searchType.setItems(types);
		searchType.setEditable(true);
		searchType.setPlaceholder(new Label("File Type"));

		searchCategory.setItems(categories);
		searchCategory.setEditable(true);
		searchCategory.setPlaceholder(new Label("Category"));
	}
	
	@FXML
	public void initialize() {
		this.setHeader();
		this.setDataView();
		this.setDirectoryView();
		
		gmarkLogo.setImage(new Image("file:resources/gmarkLogo.png"));
		
		accordionList.setExpandedPane(fileListAccordion);
		accordionList.expandedPaneProperty().addListener((property,oldValue,newValue)->{
			
			// OldValue ==> which is closed
			
			// newValue ==> which is open
			
			//System.out.println("Old :::::: " + oldValue );
			//System.out.println("New  ::::: " + newValue);
			if(oldValue != null)
			{
				if (oldValue.equals(optionsAccordion)) accordionList.setExpandedPane(fileListAccordion);
				if (oldValue.equals(fileListAccordion)) accordionList.setExpandedPane(optionsAccordion);
			}
			

			
			
		});
		
		
		message.setText("For feedback and suggestions please mailto Onlysumitg@gmail.com.");
	}
	
	

	private void setDataView() {
		fileList.setPlaceholder(new Label("No file found here!"));
		fileList.setStyle("-fx-font-size: 10pt");
		//fileList.setStyle( "-fx-alignment: CENTER-LEFT;");
		fileList.setItems(data);
		fileList.setEditable(true);
		
 
		
		
	 
		
		fileLocation.setCellValueFactory(new PropertyValueFactory<FileEntry, String>("actualName"));
		fileLocation.setPrefWidth(fileTag.getPrefWidth());
		fileLocation.setCellFactory(callback->{
						return new  TableCell<FileEntry, String>() {
							@Override
							public void updateItem(String catvalue, boolean empty) {
								if (catvalue==null) return ;
								super.updateItem(catvalue, empty);
								this.setText(catvalue);
								this.addEventFilter(MouseEvent.MOUSE_CLICKED, mouseEvent->{
									if(mouseEvent.getClickCount()>1 && mouseEvent.getButton() == MouseButton.PRIMARY)
									{
											try{java.awt.Desktop.getDesktop().open(new File(catvalue).getParentFile());}
											catch(Exception e){		}
									}
								});
							}
						};
		});
		
		

		fileName.setCellValueFactory(new PropertyValueFactory<FileEntry, Path>("actualPath"));
		fileName.setCellFactory(c -> new FileIconCell());

		fileCreated.setCellValueFactory(new PropertyValueFactory<FileEntry, Date>("dateCreated"));
		fileModified.setCellValueFactory(new PropertyValueFactory<FileEntry, Date>("dateModified"));

		fileType.setCellValueFactory(new PropertyValueFactory<FileEntry, String>("type"));

		fileCategory.setCellValueFactory(new PropertyValueFactory<FileEntry, String>("category"));
		fileCategory.setCellFactory(new Callback<TableColumn<FileEntry, String>, TableCell<FileEntry, String>>() {

			@Override
			public TableCell<FileEntry, String> call(TableColumn<FileEntry, String> tc) {
				ComboBoxTableCell<FileEntry, String> ctc = new ComboBoxTableCell<FileEntry, String>(categories) {
					@Override
					public void updateItem(String catvalue, boolean empty) {
						super.updateItem(catvalue, empty);

						if (catvalue != null) {
							setText(catvalue.toUpperCase());
						}
					}
				};

				ctc.setComboBoxEditable(true);
				ctc.textProperty().addListener((ov, oldValue, newValue) -> {
					if (newValue != null)
						ctc.setText(newValue.toUpperCase());
				});
				return ctc;
			}

		});

		fileCategory.setOnEditCommit(new EventHandler<CellEditEvent<FileEntry, String>>() {
			@Override
			public void handle(CellEditEvent<FileEntry, String> t) {
				FileEntry f = (FileEntry) t.getTableView().getItems().get(t.getTablePosition().getRow());
				f.setCategory(t.getNewValue().toUpperCase());
				if (categories.indexOf(t.getNewValue()) == -1)
					categories.add(t.getNewValue().toUpperCase());
				if (types.indexOf(f.getType()) == -1)
					types.add(f.getType());
				FileEntryDAO.save(f);
			}
		});

		// fileType.setGraphic(new ImageView(fileImage));

		fileTag.setCellValueFactory(new PropertyValueFactory<FileEntry, String>("tag"));
		fileTag.setCellFactory(TextFieldTableCell.forTableColumn());

		fileTag.setOnEditCommit(new EventHandler<CellEditEvent<FileEntry, String>>() {
			@Override
			public void handle(CellEditEvent<FileEntry, String> t) {
				FileEntry f = (FileEntry) t.getTableView().getItems().get(t.getTablePosition().getRow());
				f.setTag(t.getNewValue());
				FileEntryDAO.save(f);
				if (types.indexOf(f.getType()) == -1)
					types.add(f.getType());
			}
		});

	}

	private void setDirectoryView() {

		treeView.setStyle("-fx-font-size: 10pt");
		DirectoryTree.getDirectoryTree(treeView);
		
        treeView.setOnMouseClicked(e->{
         
        		 
        	//System.out.println("]]]]] MOUSE CLICKED [[[[[[[");
        	FilePathTreeItem selectedItem = (FilePathTreeItem)treeView.getSelectionModel().getSelectedItem();
        	if(selectedItem.isExpanded())
        	{
        		selectedItem.setExpanded(false);
        	}
        	else
        	{
        		selectedItem.setExpanded(true);
        	}
        	//selectedItem.loadChildren();
        	 
        });

		treeView.getSelectionModel().selectedItemProperty().addListener((observable, oldValue, newValue) -> {

			//System.out.println("]]]]] value change [[[[[[[");
			FilePathTreeItem selectedItem = (FilePathTreeItem) newValue;
 
			data.clear();
			fileList.refresh();
			message.clear();
			fileList.getColumns().remove(fileLocation);

			if (!selectedItem.isDummy())
			{
				//FilePathTreeItem selected = (FilePathTreeItem) treeView.getSelectionModel().getSelectedItem();
				selectedItem.loadChildren();
			//	selectedItem.setExpanded(true);
				
				FileList.load(data, selectedItem.getActualPath());
			    message.setText(selectedItem.getFullPath().toString());
			  
			}
			else if (selectedItem.getFullPath().equals("Search Result")) {
				fileList.getColumns().add(fileLocation);
				data.addAll(searchData);
				if (searchData.size() > 0)
					message.setText("Showing last search results.");

			}

		});
	}

	@FXML
	public void openLink(Event e) {
		try{
		 java.awt.Desktop.getDesktop().browse(new URI("HTTP://www.gmark.co.in"));
		}catch(Exception exp)
		{
			
			System.out.println("Error clic::"+ exp);
		}
	}
	
	
	@FXML
	public void copyEmailToClipboard(Event e)
	{
		message.clear();
		final Clipboard clipboard = Clipboard.getSystemClipboard();
		final ClipboardContent content = new ClipboardContent();
		content.putString("onlysumitg@gmail.com");
		clipboard.setContent(content);
		message.setText("Email address copied to clipboard.");
	}

	
	private void pointToTree(String location)
	{
		ArrayList<String> locations = new BreakPath().breakPath(location);
		TreeItem<String> lastValue = null;
		for(String loc : locations)
		{
			 
			ObservableList<TreeItem<String>> items = null;
			
			if(lastValue==null)
			 items = treeView.getRoot().getChildren();
			else
				items= lastValue.getChildren();
			for(TreeItem<String> item : items)
			{
				FilePathTreeItem temp = (FilePathTreeItem)item;
				
				if(temp.getFullPath().equalsIgnoreCase(loc)) 
				{
					lastValue = temp;
					 
					treeView.getSelectionModel().select(item);
				}	
				
			}
		}
	}
	
	
	@FXML
	public void searchClick(Event e) {

		if (searchBox.getText() != null && searchBox.getText().startsWith("@"))
		{
			pointToTree(searchBox.getText().substring(1).trim());
			return;
		}
			
		
		message.setText("Looking for files.....");
		treeView.getSelectionModel().select(1);
		String query = "";
		String catagory = "";
		String type = "";

		if (searchBox.getText() != null)
			query = searchBox.getText().trim();
		if (searchCategory.getValue() != null)
			catagory = searchCategory.getValue().trim().toUpperCase();
		if (searchType.getValue() != null)
			type = searchType.getValue().trim().toUpperCase();

		if (!query.isEmpty() || !catagory.isEmpty() || !type.isEmpty()) {
			data.clear();
			searchData.clear();
			fileList.refresh();
			fileList.getColumns().remove(fileLocation);
			fileList.getColumns().add(fileLocation);
			FileList.searchAndLoad(searchData, query.toUpperCase(), catagory.toUpperCase(), type.toUpperCase());
			data.addAll(searchData);
			//message.setText("Total " + searchData.size() + " file(s) found based on your search.");

		}
		message.setText("Total " + searchData.size() + " file(s) found based on your search.");
	}
	
	@FXML
	public void searchMouseClick(MouseEvent e)
	{
		if (e.getButton() == MouseButton.PRIMARY) 
		{
			searchClick(e);
		}
		else if (e.getButton() == MouseButton.SECONDARY)
		{
			treeView.getSelectionModel().select(1);
			data.clear();
			searchData.clear();
			fileList.refresh();
			fileList.getColumns().remove(fileLocation);
			fileList.getColumns().add(fileLocation);
			searchBox.clear();
			searchCategory.getSelectionModel().clearSelection();
			searchType.getSelectionModel().clearSelection();
		}
		
	}

}
