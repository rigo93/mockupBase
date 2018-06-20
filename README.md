(function () {
    'use strict';
	
var handDrawn = angular.module('handDrawn',['servoy']).directive('base', function($timeout, $svyProperties) {
    return {
      restrict: 'E',
      scope: {
    	  model: '=svyModel',
		  svyServoyapi: '=svyServoyapi'
      },
      controller: function($scope, $element, $attrs, $window) {

    	// cache the canvas
			var paper;
			$scope.paperModel;
			var margin = 2;
			var hdWidth;
			var hdHeight;
			//check if resized, if true run resize
			function checkSize() {
				var element =  $element.parent();
				var product = element.width() + element.height();
				if ($scope.height !== product) {
					$scope.height = product;
					resize();
				}
			}

			this.startListener = function(){
			//create observer to redraw on resize, when in developer
			if ($scope.svyServoyapi.isInDesigner()) {

				// Listen to changes on the elements in the page that affect layout
				var observer = new MutationObserver(checkSize);
			    observer.observe($element.parent()[0], {
			      attributes: true,
			      childList: false,
			      characterData: false,
			      subtree: false
			    });
			}

			$scope.$watch('model.textLocation', function(newValue, oldValue) {
				resize();
			}, true);

			$scope.$watch('model.displayText', function(newValue, oldValue) {
				resize();
			}, true);


			$scope.$watch('model.size.width', function(newValue, oldValue) {
					resize();
				}, true);

			$scope.$watch('model.size.height', function(newValue, oldValue) {
				resize();
			}, true);


			$scope.$watch('model.fontType', function(newValue, oldValue) {
				resize();
			}, true);

			$scope.$watch('model.textHorizontalAlignment', function(newValue, oldValue) {
				resize();
			}, true);


			$scope.$watch('model.textVerticalAlignment', function(newValue, oldValue) {
				resize();
			}, true);
			}

			$scope.drawElement = function(){
				$scope.paperModel.drawnRect(
				margin,
				margin,
				hdWidth-2*margin,
				hdHeight-2*margin,
				1.5
			).attr({ stroke: "#000", "stroke-width": margin });
			//draw cross in rectangle
			    $scope.paperModel.drawnLine(2, 2, hdWidth-3, hdHeight-3, 2).attr({ stroke: "#000", "stroke-width": 2 });
			    $scope.paperModel.drawnLine(hdWidth -3, 2, 2, hdHeight-2,2).attr({ stroke: "#000", "stroke-width": 2 });
			}


			this.paint = function(targetElement) {
				var targetElement = $element.find(targetElement)[0];
				var currentElement = $element.parent();
				checkWrapper(currentElement, targetElement);
				drawElement();
				//paperRect.setViewBox(0, 0, hdWidth+ 'px', hdHeight+ 'px', true);
				setText();
				paper = $scope.paperModel;
			}


			function setText(){

				// fix the code in case fontType is undefined
				if($scope.model.fontType !== null){
					$scope.model.fontType = "Times New Roman";
				}
				//if(fontType === null){
				//	var fontType = "Times New Roman";
				//}
				switch($scope.model.textHorizontalAlignment + "|" + $scope.model.textVerticalAlignment) {
				    case "Left|Top":
				    $scope.$scope.paperModel.text($scope.model.textLocation.x + margin *2,$scope.model.textLocation.y + parseInt($scope.model.fontType.fontSize)/2+ margin *2, $scope.model.displayText  ).attr({'text-anchor': 'start', 'font-family': $scope.model.fontType.fontFamily, 'font-size': $scope.model.fontType.fontSize })
				        break;
				    case "Center|Top":
				    $scope.paperModel.text($scope.model.textLocation.x + hdWidth/2,$scope.model.textLocation.y + parseInt($scope.model.fontType.fontSize)/2+ margin *2, $scope.model.displayText  ).attr({'text-anchor': 'middle', 'font-family': $scope.model.fontType.fontFamily, 'font-size': $scope.model.fontType.fontSize })
				        break;
				    case "Right|Top":
				    $scope.paperModel.text($scope.model.textLocation.x + hdWidth - margin *2,$scope.model.textLocation.y + parseInt($scope.model.fontType.fontSize)/2+ margin *2, $scope.model.displayText  ).attr({'text-anchor': 'end', 'font-family': $scope.model.fontType.fontFamily, 'font-size': $scope.model.fontType.fontSize })
				        break;
				    case "Left|Middle":
				    $scope.paperModel.text($scope.model.textLocation.x,$scope.model.textLocation.y + hdHeight/2 , $scope.model.displayText  ).attr({'text-anchor': 'start', 'font-family': $scope.model.fontType.fontFamily, 'font-size': $scope.model.fontType.fontSize })
				    	break;
			        case "Center|Middle":
			        //paperRect.text($scope.model.textLocation.x + hdWidth/2 ,$scope.model.textLocation.y + hdHeight/2 , $scope.model.displayText  ).attr({'text-anchor': 'middle', 'font-family': $scope.model.fontType.fontFamily, 'font-size': $scope.model.fontType.fontSize })
			        	break;
			        case "Right|Middle":
			        $scope.paperModel.text($scope.model.textLocation.x + hdWidth - margin *2,$scope.model.textLocation.y + hdHeight/2 , $scope.model.displayText  ).attr({'text-anchor': 'end', 'font-family': $scope.model.fontType.fontFamily, 'font-size': $scope.model.fontType.fontSize })
			        	break;
				    case "Left|Bottom":
				    $scope.paperModel.text($scope.model.textLocation.x,$scope.model.textLocation.y + hdHeight - parseInt($scope.model.fontType.fontSize)/2 , $scope.model.displayText  ).attr({'text-anchor': 'start', 'font-family': $scope.model.fontType.fontFamily, 'font-size': $scope.model.fontType.fontSize })
				    	break;
				    case "Center|Bottom":
				    $scope.paperModel.text($scope.model.textLocation.x + hdWidth/2,$scope.model.textLocation.y + hdHeight - parseInt($scope.model.fontType.fontSize)/2 , $scope.model.displayText  ).attr({'text-anchor': 'middle', 'font-family': $scope.model.fontType.fontFamily, 'font-size': $scope.model.fontType.fontSize })
				    	break;
				    case "Right|Bottom":
				    $scope.paperModel.text($scope.model.textLocation.x + hdWidth - margin *2,$scope.model.textLocation.y + hdHeight - parseInt($scope.model.fontType.fontSize)/2, $scope.model.displayText  ).attr({'text-anchor': 'end', 'font-family': $scope.model.fontType.fontFamily, 'font-size': $scope.model.fontType.fontSize })
			        break;
				}
			}



			function checkWrapper(currentElement, targetElement){
				if (currentElement.hasClass('.svy-wrapper')) {
					//responsive
					var currentElement = $element.parent();
					hdWidth = $scope.model.size.width;
					hdHeight = $scope.model.size.height;
					var myNewDiv = "<div id='omg'></div>";
					myNewDiv = $( "#omg" );
					myNewDiv.width(hdWidth);
					myNewDiv.heigth(hdHeight);
					myNewDiv.css("display", "inline-block");
					currentElement.append(myNewDiv);
					$scope.paperModel = Raphael(currentElement, hdWidth+ 'px', hdHeight+ 'px');
					//paperRect.setSize(hdWidth + 'px', hdHeight + 'px');
				} else {
					//anchored
					hdWidth = currentElement.width();
					hdHeight = currentElement.height();
					$scope.paperModel = Raphael(targetElement, '100%', hdHeight + 'px');
				}
			}

			function resize() {
				// resize the paper if already drawn
				if (paper) {
					paper.clear();
					paint();
				}
			}
      },
      link: function(scope, element, attrs, baseCtrl){
    	  baseCtrl.paint('.svy-rectangle');
    	  baseCtrl. startListener();
      }
    };
  })
  })();
  
